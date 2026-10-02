---
name: google-play-app-setup
description: Take an Android app from "builds locally" to "created in Play Console with every App content declaration, the store listing and a signed AAB ready for a closed test" - target-SDK and CI readiness, package name, privacy policy hosting, keeping the developer's personal identity off public surfaces, and the declarations (Data safety, content rating, target audience, sign-in details, ads, advertising ID). Use when the user says "help me set up a Play Console profile for the app", "create the app in Google Play", "fill in App content / Data safety / content rating", "write the store listing", or "get this ready for a closed test".
---

# Google Play: first-time app setup

Get a new app created in Play Console with all policy declarations answered, a store listing, and a signed bundle that a CI pipeline really produces. This is the stage *before* [google-play-release](../google-play-release/SKILL.md) (uploads, service accounts, in-app products). Read [REFERENCE.md](REFERENCE.md) first: it lists the traps hit in practice and the exact answers that passed.

## When to use

- The developer account exists (or is about to) and this app has no Play Console entry yet, or its App content / store listing is incomplete.
- Not for automated uploads or in-app products: use google-play-release.

## Process

1. **Sync and read.** Pull the repository. Check whether CI is actually green; release pipelines are often broken for days unnoticed. Read the manifest, permissions, dependencies and any network code. The declarations must describe what the code does, not what you assume.
2. **Learn the publisher's conventions.** Open the developer account (it may be on a second signed-in Google account) and read the existing apps: their package-name prefix, website, privacy-policy host and contact e-mail. Follow them.
3. **Ask about identity before anything public.** Ask whether the human's personal name may appear anywhere. If not, every public surface needs a publisher-owned home: website, privacy policy, contact e-mail, links in the description and reviewer notes, download pages for companion apps, and the binaries themselves.
4. **Make the build Play-ready.**
   - Target the currently required API level.
   - Handle that level's behavior changes, such as enforced edge-to-edge.
   - Check that native libraries are aligned for 16 KB pages.
   - Produce a signed AAB from CI with an upload key the human generates. Give them a script, so the password never passes through you.
   - Verify by building locally, then inspect the bundle's target SDK and its signing certificate.
5. **Decide the package name with the human.** The Create app form asks for it and it can never change. If it differs from the code's `applicationId`, change only `applicationId`.
6. **Privacy policy.** Write it from the verified data flows: what is stored locally, what leaves the device and to whom, and why each permission is needed. Host it on the publisher's site, one page per app. Check that the URL returns 200 and contains no personal name.
7. **Create the app.** This means accepting the Developer Program Policies and US export declarations, so get explicit consent first. Note whether the account is personal: new personal accounts need a 14-day closed test with at least 12 testers before production.
8. **App content.** Work through every declaration using the answer rules in REFERENCE.md. Content rating requires accepting the IARC terms, so ask first. After each save, confirm the "Change saved" message. Finish when the overview says nothing needs attention.
9. **Store listing.**
   - Write the default language and its translations within the limits: name 30, short description 80, full description 4000.
   - Add translations through *Select languages*. Never use the paid *Purchase translations*.
   - Set the category and contact details in Store settings.
   - Graphics (icon, feature graphic, screenshots) come from the human.
10. **Verify and hand over.** Re-read each saved value from Play after a reload, and scan public pages and release binaries for the personal name. Then list what only the human can do.

## Output format

A table of what is now set in Play Console (section, value). Then what was verified and how. Then a numbered list of human-only steps: key generation, tokens or scopes, graphics, testers. Give exact menu paths and commands for each.

## Anti-patterns to avoid

- Answering Data safety or the content rating from the app's genre instead of its code and merged manifest.
- Picking the package name, accepting terms, or entering contact data without asking.
- Hosting the privacy policy or downloads on a personal account when the publisher identity must stay separate. Transferring a repo does not hide author names in its git history.
- Typing into a hidden or minimized browser window and assuming it worked. Read the value back.
- Treating "CI exists" as "CI works".
