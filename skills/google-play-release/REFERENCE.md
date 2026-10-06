# Google Play release: field notes and API recipes

Companion to [SKILL.md](SKILL.md). Everything here was hit in practice while
setting up in-app products and automated uploads for a Godot 4 Android game
(Rune Survivor, `com.ogbar.runesurvivor`, October 2026) and for a native
Kotlin/Compose app with product flavours (Picture Dictionary,
`com.ogbarlabs.picturedictionary`, October 2026). The same patterns apply to
any Android build system.

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
| 11 | Build step "stuck" at the export for 30+ minutes, ~0% CPU, although the .aab may already exist | **Main cause:** the Gradle daemon the export starts inherits the step's stdout/stderr pipe and outlives the build, so the console wrapper (Godot's `_console.exe`, a CI step, a build agent) waits for pipe EOF forever. Low free RAM (0.6 GB, other apps, emulator, idle daemons) made it look like a memory stall at first | Run CI/agent builds with `GRADLE_OPTS=-Dorg.gradle.daemon=false` (verified: the command returned 6 s after the bundle instead of never). Still check free memory before Gradle builds. Check whether the output file exists before blaming slowness. Don't kill other people's processes; report them |
| 12 | Console automation flaky | Angular UI: table rows missing from the accessibility tree until rendered, page-text extraction returns fragments, a zoom action left the viewport stuck at 360×112 | Prefer the API. In the UI, read state with page scripts, open a fresh tab when the viewport breaks, and use element refs over coordinates |
| 13 | Browser file upload refused the PEM in the home folder | Upload is limited to folders the agent session may read | Copy the *public* certificate into the session's working/scratch folder first. Never do this with private keys |
| 14 | Python script crashed printing prices | Windows console code page (cp1250) can't encode `→`, `—` | Keep CLI output ASCII, or force UTF-8 output |
| 15 | PowerShell step failed to parse | `"$name: text"` is parsed as a scope-qualified variable | Write `"${name}: text"` |
| 16 | Agent sessions blocked on account-level changes | Safety guards stop agents from submitting key resets, killing processes, or reading credential files | Prepare everything, then hand the human an exact, short checklist |
| 17 | Console wizard never advances, screenshots time out after 30 s, `read_page` reports `Viewport: 0x0` | The Chrome window is minimised or in the background, so the page's animation frames are paused | Ask the human to bring the Chrome window to the front before any Console automation. The extension also drops its tab group now and then: call `tabs_context_mcp` again for the new tab id |
| 18 | Browser-extension file upload refuses the .aab | The extension's `file_upload` is capped at 10 MB; a bundle with bundled media is 70+ MB. Fetching it from a local web server inside the page hangs too (the browser's local-network permission prompt waits for the human) | Prepare the release form (name, notes), then have the human click *Upload* and pick the .aab. Don't try to smuggle a signed bundle through a public host |
| 19 | API upload to a brand-new app is refused or the bundle is unknown to Play | The first bundle of a new app is what enrolls Play App Signing, and only the Console flow does that | Upload the first .aab by hand in the Console; let the pipeline take over from the second one |
| 20 | Auto-mode classifier denies building the signed bundle or editing `~/.gradle/gradle.properties` although the chat approved it | The classifier ignores chat approval for signing and credential-adjacent actions | Don't look for another route. Give the human the exact permission rule to paste into `.claude/settings.local.json` (`Bash(./gradlew :app:bundlePlayRelease)`) or the exact line to add to `gradle.properties`, and continue once it's saved |
| 21 | forge build fails with `nothing matched …/apk/release/*.apk` although `forge.yaml` in the repo is right | The hub keeps its own copy of the recipe and builds from it | After editing `forge.yaml`, run `forge app config <app> forge.yaml`, then rebuild |
| 22 | Build node vanished from the hub; `forge.exe` and its `forge-agent` task are gone | Windows Defender quarantined an unsigned Go binary as `Trojan:Win32/…!ml` (heuristic false positive) | Add the exclusion for the install folder (`Add-MpPreference -ExclusionPath …\Programs\forge` and `~\.forge`) *before* restoring (`MpCmdRun.exe -Restore -Name "<threat>" -All`). Restoring two old detections can leave two conflicting scheduled tasks; delete the task and `forge agent install` again. `agent status` only reports the installed service, so check the hub (`forge node ls`) for the real state |
| 23 | Publishing overview: "Submit changes" is disabled, red banner *1 issue affects all of your changes: Missing sign in details* | Play's pre-launch crawler screenshotted a gate (here a first-run parent-code screen) and the "Sign in details" declaration (formerly App access) said *No*. An in-app purchase also counts as restricted access | Set the declaration to *Yes*, add one entry (name, username, password, instructions) that gets a reviewer past the gate, and create promo codes (Monetize with Play → Promo codes → one-time product → *Download codes*, a CSV) so the reviewer can unlock paid content. The mandatory checkbox "provide full access … including premium or paid content" must then be true, so don't tick it without codes. Switch the *share with Google testing* toggle off, or the crawler burns single-use codes. The instruction field is capped at 500 characters |
| 24 | In the sign-in dialog the Add button says *Your changes couldn't be saved*, though the text is visible | `form_input` set the textarea but the form never saw an input event | Click the field, `ctrl+a`, and type the text for real; then Add |

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

## License testing (test purchases instead of real money)

Settings → *License testing* (`/console/u/N/developers/<id>/license-tester`) is **account-wide**, not per app. Select one or more email lists (create them under Settings → *Email lists*; a list called `just-me` holds the owner's own accounts) and save. Everyone in a selected list buys with Play's test instruments: the purchase sheet says it is a test order, nothing is charged, and test purchases are cancelled automatically after a few minutes. The *License response* dropdown only drives the old licensing library (LVL), so leave it at `RESPOND_NORMALLY` for billing tests.

A tester still has to be able to install the app from Play: put the account on a testing track's tester list too (an email list with the same account is fine) and install through the opt-in link, not a sideload.

Before changing it, open the page: the list may already be selected (it was, for `just-me`), and since the setting is shared, adding lists affects every app of the account.

## Native Android (Gradle) pipeline

- Gradle Play Publisher (`com.github.triplet.play`) on the `play` product flavour: `play { enabled.set(false) }` globally, `android.playConfigs { register("play") { enabled.set(true) } }` for the flavour, task `publishPlayReleaseBundle`. Track, draft flag and service-account path come from machine settings (environment or `~/.gradle/gradle.properties`), never from the repo.
- Fail early when the upload key is missing (`check(uploadProperties != null)` in a `doFirst`): an unsigned bundle is refused by Play with a much less plain message.
- Flavour application ids: set the flavour's `applicationId` in `productFlavors`; with only a build type changing the id, apply the plugin *after* the `androidComponents` block, or it reads the old id.
- A forge recipe for it (`tools/<app>-play.forge.yaml`) needs the `play-upload` requirement so it only runs on the labelled machine; register it with `forge app add <name> --repo … --ref … --config tools/<file>`.
- A one-time product can be created in the console in about a dozen clicks: Monetize with Play → One-time products → Create. Product ID, name, description, purchase option ID `buy`, *Set prices → Bulk edit prices* for all regions with the home currency (the rest are converted), then *Activate*. The BILLING-permission gate (row 2) must be passed first.

## Build-machine pattern (secrets outside the repo)

When the build runner can't receive secrets, put them on one machine and route the job there by a label (e.g. `play-upload`). Keep one properties file outside the repo with: keystore path, keystore password, key alias, service-account JSON path, track, draft flag. The human writes the password into it; the agent may create the file with a placeholder.
