# Google Play release: field notes and API recipes

Companion to [SKILL.md](SKILL.md). Everything here was hit in practice while
setting up in-app products and automated uploads for a Godot 4 Android game
(Rune Survivor, `com.ogbar.runesurvivor`, October 2026). The same patterns
apply to any Android build system.

## Problems we ran into, and what fixed them

| # | Symptom | Cause | Fix / rule |
|---|---|---|---|
| 1 | Play Console opens "Creating a developer account" | The developer account belongs to a *second* Google account in the browser | Try `/console/u/1/…`, `/u/2/…` before deciding there is no account |
| 2 | Products page: "To add one-time products, you need to add the BILLING permission to your APK"; API: `Can't create product. To fix, request billing permission.` | No bundle with `com.android.vending.BILLING` has ever been uploaded | Ship the billing library, upload one bundle to any track (internal is fine), then create products. UI and API enforce the same gate |
| 3 | Billing plugin breaks the plain APK export | Godot's Play Billing plugin (v2 Android plugin) pulls `billing-ktx` from Maven, so it only works with the Gradle build | Keep the plugin disabled in the committed project file and enable it at build time only in the Play (Gradle/.aab) pipeline |
| 4 | Unsure whether the plugin really made it into the bundle | Plugins can be silently skipped | Fail the build unless `base/manifest/AndroidManifest.xml` inside the .aab contains the bytes `com.android.vending.BILLING` (the manifest is protobuf, but permission names are plain UTF-8) |
| 5 | Upload key exists only in a CI secret | CI secrets can't be read back | Upload key reset: new keystore → `keytool -export -rfc … -file cert.pem` → App integrity → *Request upload key reset* → reason + PEM. 1–2 days. The human submits it |
| 6 | Godot release signing fails or uses the wrong password | Godot has one password field for store and key | Key password must equal store password; say so when the human creates the key |
| 7 | Release rejected on a never-published app | Draft apps only accept `status: draft` releases | Make draft configurable, or retry the track update as draft when Play's error mentions a draft app |
| 8 | Commit rejected: changes can't be sent for review automatically | Account/app state | Retry commit with `changesNotSentForReview=true` |
| 9 | Bundle rejected for version code | Another pipeline (e.g. CI run numbers) already used a higher code, or will later | Read existing codes first (`edits.bundles.list`). Pick a scheme that only grows and stays above every other pipeline's, e.g. `tag_n * 1000 + commits_since_tag` |
| 10 | Duplicate work: an upload recipe "already exists" on the build hub | Local `main` was stale; another session had merged the recipe that morning | Fetch and read `origin/main` before building anything |
| 11 | Gradle export sat at 0% CPU for 30+ minutes | Workstation had 0.6 GB free RAM, commit limit nearly exhausted (other apps, emulator, idle Gradle daemons) | Check free memory before Gradle builds. Leftover daemons from your own earlier builds add up. Don't kill other people's processes; report them |
| 12 | Console automation flaky | Angular UI: table rows missing from the accessibility tree until rendered, page-text extraction returns fragments, a zoom action left the viewport stuck at 360×112 | Prefer the API. In the UI, read state with page scripts, open a fresh tab when the viewport breaks, and use element refs over coordinates |
| 13 | Browser file upload refused the PEM in the home folder | Upload is limited to folders the agent session may read | Copy the *public* certificate into the session's working/scratch folder first. Never do this with private keys |
| 14 | Python script crashed printing prices | Windows console code page (cp1250) can't encode `→`, `—` | Keep CLI output ASCII, or force UTF-8 output |
| 15 | PowerShell step failed to parse | `"$name: text"` is parsed as a scope-qualified variable | Write `"${name}: text"` |
| 16 | Agent sessions blocked on account-level changes | Safety guards stop agents from submitting key resets, killing processes, or reading credential files | Prepare everything, then hand the human an exact, short checklist |

## Service account permissions (app level)

- *View app information* — read tracks, bundles.
- *Release apps to testing tracks* — upload and roll out to internal/closed/open.
- *Manage store presence* — "manage in-app products" is inside this one.
- Production releases need *Release to production…* as well. Grant only when wanted.

## Upload (Edits API, `androidpublisher/v3`)

1. `POST applications/{pkg}/edits` → `id`
2. `POST upload/androidpublisher/v3/applications/{pkg}/edits/{id}/bundles?uploadType=media` (body: the .aab, `application/octet-stream`; allow ~15 min timeout)
3. `PUT …/edits/{id}/tracks/{track}` with `{"track": t, "releases": [{"name": n, "versionCodes": ["123"], "status": "completed"|"draft"}]}`
4. `POST …/edits/{id}:commit` (optionally `?changesNotSentForReview=true`)
5. On any failure: `DELETE …/edits/{id}`

## One-time products (new monetization API)

1. `POST applications/{pkg}/pricing:convertRegionPrices` with `{"price": {"currencyCode": "EUR", "units": "0", "nanos": 830000000}}`
   - Input is **tax-exclusive**. Output per region is tax-inclusive, plus `regionVersion.version` and `convertedOtherRegionsPrice`.
   - For a gross €0.99 in Germany, send net 0.83 (gross ÷ 1.19). Results: DE €0.99, HU 389 Ft, US $0.89; 174 regions.
2. `PATCH applications/{pkg}/onetimeproducts/{id}?allowMissing=true&updateMask=listings,purchaseOptions&regionsVersion.version=<v>`
   body: `listings[{languageCode,title≤55,description≤200}]`, `purchaseOptions[{purchaseOptionId:"buy", buyOption:{legacyCompatible:true}, regionalPricingAndAvailabilityConfigs[{regionCode, price, availability:"AVAILABLE"}], newRegionsConfig{usdPrice,eurPrice,availability}}]`
3. `POST applications/{pkg}/oneTimeProducts/{id}/purchaseOptions:batchUpdateStates` with `activatePurchaseOptionRequest` (new options start as DRAFT).
4. `GET …/onetimeproducts/{id}` before creating. Skip existing products so prices edited in the console are never overwritten.

`legacyCompatible: true` is what older billing libraries and plugins see as `oneTimePurchaseOfferDetails`. Consumable vs non-consumable is decided by the app (consume vs acknowledge), not by Play.

## Build-machine pattern (secrets outside the repo)

When the build runner can't receive secrets, put them on one machine and route the job there by a label (e.g. `play-upload`). Keep one properties file outside the repo with: keystore path, keystore password, key alias, service-account JSON path, track, draft flag. The human writes the password into it; the agent may create the file with a placeholder.
