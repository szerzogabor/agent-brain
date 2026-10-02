# Google Play first-time setup: field notes

Companion to [SKILL.md](SKILL.md). Everything here was hit in practice while taking a Kotlin/Compose Android app (ControlDeck, `com.ogbar.controldeck`, LAN-only, with a Windows companion app) from "builds locally" to "created in Play Console, App content complete, signed AAB from CI", October 2026. See also [google-play-release/REFERENCE.md](../google-play-release/REFERENCE.md) for upload and in-app-product traps.

## Step by step (what actually worked, in order)

1. Pull the repository. List the recent CI runs and their conclusions. Read the build config (`applicationId`, `targetSdk`, signing), the manifest, and the dependency list.
2. Open Play Console under every signed-in account (`/console/u/0/`, `/u/1/` …) until the developer account appears. Note:
   - **Personal or organization.** A personal account needs the 12-tester, 14-day closed test.
   - **The other apps' conventions:**
     - package prefix (e.g. `com.<publisher>.*`)
     - website (e.g. `https://<publisher>.github.io`)
     - privacy URL pattern (e.g. `/<app>/privacy.html`)
     - contact e-mail
3. Ask the human:
   - **package name;**
   - **whether their personal name may appear publicly** (assume not if they created a separate publisher account);
   - **which account owns public pages and downloads.**
4. Fix build readiness, then verify locally:
   - Build tests, lint, a debug APK and a release AAB.
   - Inspect the bundle: `aapt2 dump badging` on the APK for `targetSdkVersion`, the 16 KB ELF alignment check below, and `keytool -printcert -jarfile app.aab` against the upload keystore.
5. Fix CI so `main` really produces a signed AAB. Push only after the human has the needed token scopes.
6. Write the privacy policy page in the publisher site's existing style. Push it under the publisher identity and confirm it returns HTTP 200.
7. Create the app:
   - name;
   - package name, after "Check availability";
   - default language;
   - App or game;
   - Free or paid. A free app can never become paid later; in-app products remain possible.
   - Two declaration checkboxes, ticked only after the human consents.
8. App content: complete all 10 declarations (answers below).
9. Store listing:
   - texts in the default language;
   - *Manage translations → Select languages* for each extra language;
   - Store settings: category plus contact e-mail and website.
10. Hand over the graphics, testers and remaining secrets to the human.

## Answer rules for App content (and why)

| Declaration | Answer that passed | Rule |
|---|---|---|
| Privacy policy | Publisher-site URL | Must be public, non-PDF, specific to the app |
| Ads | No | Only if no ad SDK is in the dependency tree |
| Sign in details (formerly App access) | **Yes**, restricted, with reviewer instructions | The "Yes" text explicitly lists QR codes, one-time PINs and "actions on another device". Pairing flows count even without accounts. Leave username/password empty; instructions max **500 chars**; tick "full access" only if nothing is paywalled |
| Advertising ID | No | Verify the **merged** release manifest has no `com.google.android.gms.permission.AD_ID` (libraries can add it) |
| Government / Financial / Health | No / "no financial features" / "no health features" | Financial and Health are 2-step forms: tick the "none" box, Next, Save |
| Target audience | 18 and over | Selecting only 18+ collapses the remaining steps to a summary; avoid under-13 groups unless the app is built for the Families policy |
| Content rating | Category "All Other App Types", every question No | Requires a contact e-mail and accepting the **IARC Terms of Use** (ask the human). Questions appear progressively; Next stays disabled until you Save once. Result: PEGI 3 / ESRB Everyone |
| Data safety | "Does your app collect or share…" → **No** | "Collection" means data leaves the device to the developer or a third party. Peer-to-peer traffic between the user's own devices, with no backend, analytics or crash reporting, is not collection. Re-answer the moment any SDK is added |

## Problems we ran into, and what fixed them

| # | Symptom | Cause | Fix / rule |
|---|---|---|---|
| 1 | Console opened the "create a developer account" page | Developer account lived on the second signed-in Google account | Try `/u/1/`, `/u/2/` first |
| 2 | Create app form asked for the package name, permanent | Newer form fixes the package at creation | Decide with the human before opening the form; match the account's prefix. Changing only `applicationId` (not the Kotlin `namespace`) keeps the code untouched |
| 3 | App targeted API 34 | Play requires API 36 for new apps and updates since 2026-08-31 (extension possible to 2026-11-01) | Bump compile/target SDK. AGP 8.5 cannot do 36: moved to AGP 8.13, which needs Gradle ≥ 8.13 |
| 4 | CI Gradle version unknown, drifting | Runner image ships its own (9.x) Gradle | Pin `gradle-version` in the setup step to the version you verified locally |
| 5 | Content under status/navigation bars risk | API 35+ enforces edge-to-edge | Opt in on all versions; let the root Scaffold pad for insets and **consume** them so nested Scaffolds don't pad twice |
| 6 | 16 KB page-size requirement | Native `.so` files inside dependencies | For each arm64/x86_64 `.so` in the AAB, every ELF `PT_LOAD` segment's `p_align` must be ≥ `0x4000` (a short script reading the program headers is enough) |
| 7 | Android CI red for days | `setup-android` action's default package list includes the removed `tools` package; new cmdline-tools reject it | Set its `packages` input to `platform-tools` only |
| 8 | Release workflow failed in 0 s, "workflow file issue" | `secrets.X` used in a step's `if:`, which is invalid and kills the whole file | Expose a job env `HAS_X: ${{ secrets.X != '' }}` and test `env.HAS_X == 'true'` |
| 9 | Windows CI red | A test asserted the machine has a physical network adapter; CI VMs only have virtual ones | Accept the documented "none" result instead of asserting hardware |
| 10 | Push rejected: "refusing to allow an OAuth App to create or update workflow" | CLI token lacked the `workflow` scope | The human runs the CLI's auth refresh with the `workflow` scope; the agent can't grant it |
| 11 | Local build: no JDK 17 for `jvmToolchain(17)` | Only JDK 21 installed | Local-only Gradle init script: toolchain 21, `release`/`jvmTarget` 17; never commit it, don't download JDKs without consent |
| 12 | Upload key needed, agent must not see the password | Secret handling rules | Commit a script the human runs: random password, `keytool -genkeypair` (PKCS12, store pass = key pass), sets the four CI secrets, keeps keystore + password file outside the repo |
| 13 | Screenshots fail with "0 width", typing does nothing, dialogs don't open | Browser window minimized/hidden (`document.visibilityState == "hidden"`) | Ask the human to keep the window visible (not necessarily focused); opening a fresh tab in the agent's tab group also restored rendering. Tab IDs change afterwards, so re-list tabs |
| 14 | Typed description didn't stick | Hidden window drops key events | Set the field value through the page's form-fill mechanism, then read back the character counter (`74 / 80`) |
| 15 | "Start declaration" click only scrolled the page | Angular card button vs. inner link | Programmatically click the element whose `aria-label` is "Start <name> declaration" |
| 16 | Radio answers not registered (IARC later questions, Data safety "No") | Programmatic clicks on Material radios sometimes ignored | Click by screen coordinates after a screenshot; confirm via the section's "Completed" tick |
| 17 | Text limit errors | Sign-in instructions limit is 500 chars | Write short numbered steps; read the counter before saving |
| 18 | Unexpected extra app in the list | Another session/human created one concurrently | Read names before assuming a duplicate; never delete |
| 19 | Personal name on public pages | Website and privacy URL first pointed at a personal GitHub repo | Move every public link to the publisher site and a source-less **release-only repo** under the publisher account |
| 20 | Personal name inside release binaries | .NET PDBs embed SourceLink URLs of the source repo | Publish with `DebugType=none` and `DebugSymbols=false`; scan every file in the zip (expect false positives in localized resource DLLs, e.g. Polish "rozszerzony") |
| 21 | Repo transfer considered | Git history keeps author names | Don't transfer; make the source repo private and mirror only downloads |
| 22 | `/releases/latest` 404 on the mirror | `latest` ignores prereleases; also needs ≥ 1 commit | Mirror as a normal release; seed the repo with a README (contents API with explicit author/committer = publisher) |
| 23 | Two CLI accounts on one machine | Publisher account logged in but not active | Run single commands with that account's token in the environment; don't switch the active account. Never copy a token into a secret yourself; give the human the one-line command |
| 24 | Scripted YAML edit silently broke a `run:` block | Backslash-newline inside a non-raw script string joined the lines | Re-read files after scripted edits; prefer exact-string edits for multi-line YAML |
| 25 | APK on a public mirror | Play installs are re-signed with Google's key; a sideloaded upload-key APK clashes | Mirror only companion downloads (e.g. the Windows zip), not the Android APK |

## Facts worth remembering

- A free app cannot become paid after publishing; in-app products can be added later (BILLING permission + merchant profile, see google-play-release).
- Every App content save says "Change saved. Send for review in Publishing overview." Nothing is sent for review until a release is submitted.
- New personal accounts: production access only after a closed test with ≥ 12 opted-in testers for 14 consecutive days.
- Store listing graphics: icon 512×512 PNG; feature graphic 1024×500; ≥ 2 phone screenshots (≥ 4 at ≥ 1080 px to be eligible for promotion); tablet screenshots strongly advised for tablet-first apps.
