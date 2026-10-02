---
name: google-play-release
description: Set up Google Play Console for an Android app end to end - in-app products (one-time purchases), the BILLING permission, the upload key, a service account, and automated .aab uploads from CI or a build machine. Use when the user says "set up the missing purchase settings in Play Console", "create the in-app products", "release to Google Play", "upload the AAB automatically", "Play says I need the BILLING permission", or "we lost the upload key".
---

# Google Play release and in-app products

Get an app from "a project that builds" to "a build on a Play testing track that can sell its products", with the upload automated. The order matters: Play blocks later steps until earlier ones are done, and two of them need the human and a 1–2 day wait. Read [REFERENCE.md](REFERENCE.md) before starting. It lists the traps hit in practice and the exact API calls.

## When to use

- In-app products are missing or can't be created, or the user asks for "purchase settings" in Play Console.
- Wiring a CI job or build machine to upload bundles to a Play track.
- Not for store-listing copy or content-rating questionnaires. Those are forms the human fills in.

## Process

1. **Sync first.** Fetch the repository and read any existing Play tooling (upload scripts, CI recipes, docs). A previous session may have done half the job.
2. **Find the right account.** The developer account is often on a secondary Google account. If the console redirects to "create a developer account", try the other signed-in accounts before concluding there is none. Confirm a payments profile with a bank account exists, because paid products need it.
3. **Inspect before changing.** Through the Play Developer API (or read-only in the console), list the app's tracks, releases and version codes, and the existing service-account users. Write down the highest version code and the current upload-key fingerprint.
4. **Service account.** Reuse one if the account already has it. Otherwise create it in a Google Cloud project with the Android Publisher API enabled and keep its JSON key outside the repository. In Play Console → Users and permissions, give it access to this app: *Release apps to testing tracks*, *View app information*, and *Manage store presence* (the last one covers in-app products). There is no project-linking step any more.
5. **Upload key.** Signing needs the key Play knows as the app's upload key. If it only exists in a CI secret (unreadable) or is lost, the human generates a new key, exports its PEM certificate and requests an upload key reset in App integrity. Approval takes 1–2 days. The human must submit this form; do not attempt it yourself.
6. **Billing in the build.** Add the platform's Play Billing library or plugin. Then verify on the built .aab that the base manifest requests `com.android.vending.BILLING`. Play refuses to create products until a bundle with that permission has been uploaded to any track.
7. **Upload** through the Edits API: insert edit → upload bundle → update track → commit, and delete the edit on any failure. Use a version code above every code Play already has. Start on the internal track; never-published apps may only accept `draft` releases.
8. **Create products** from the project's product catalogue: one-time products with one legacy-compatible buy option, prices converted per region by Play's converter, then activate them. Run a dry run first and show the user the resulting prices.
9. **Testing.** Add the user's accounts under Settings → License testing, install from the testing track (not a sideload), and buy one product of each kind.
10. **Verify** each stage against Play itself (track listing, product listing), not against your script's exit code.

## Output format

Report what is now true in Play Console, what was verified and how, and a numbered list of the steps only the human can do (key reset, passwords, forms), with exact file paths and menu paths.

## Anti-patterns to avoid

- Clicking through the console for bulk work the API does reliably (30 products, uploads).
- Writing passwords or keys into the repository, CI logs or chat. Keys live on the machine or in secret stores, and the human types passwords.
- Uploading a version code scheme that later collides with another pipeline's codes.
- Starting a Gradle/AAB build on a machine with almost no free memory. Check first, because it stalls instead of failing.
- Assuming a local branch is current, or that "no products page" means a missing setting rather than a missing BILLING build.
