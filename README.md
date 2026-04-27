# Finance Tracker — QA & Bug Reports

This repository is the **public bug tracker** for the Finance Tracker mobile app. The app source code lives in a separate private repository; this repo contains **no code** — only issue templates and reporting guidelines.

If you are a QA tester, this is the right place. Welcome, and thank you for testing.

## 📦 Current test build

> **Test only the build linked here.** Reports from older builds, side-loaded variants, or unofficial copies cannot be triaged.

| Field | Value |
|---|---|
| **Platform** | Android (APK) |
| **App version** | `0.17.0` |
| **Build link** | https://expo.dev/accounts/ericocampos/projects/finance-tracker-mobile/builds/3a28620c-afd2-42be-bdcc-f5a65f381288 |
| **Build ID** | `3a28620c-afd2-42be-bdcc-f5a65f381288` |

**To install**: open the Build link above on your Android device, then tap "Install" on the Expo build page. You may need to allow "Install from unknown sources" the first time.

**When you file an Issue, paste the Build link above into the *Build link* field** so we know exactly which build you tested. Issues without a build link cannot be triaged and will be closed as `needs-info`.

This section is updated when a new build ships. **Always re-check it before testing**, and update to the latest build before filing reports.

## 📖 Read this first

**[How the app should behave](docs/app-behavior.md)** — the canonical reference for what the app is supposed to do, what the validation rules are, what the error messages should say, and what's deliberately *not* a feature yet. **Read this before planning test cases or filing your first bug.** If the build does something the doc doesn't describe, that itself is worth reporting.

## How to report a bug

1. Go to the [**Issues**](https://github.com/ericocampos/finance-tracker-bugs/issues) tab.
2. Click **New issue**.
3. Pick the template that matches what you found:
   - **Bug report** — something is broken or behaves wrong.
   - **UI / UX issue** — visual glitch, awkward flow, confusing copy.
   - **Crash report** — the app closed unexpectedly.
   - **Feedback / suggestion** — not a bug, but worth saying.
4. Fill in **every** field in the template. Reports missing reproduction steps or device info are very hard to act on.

## Before you submit

- **Search existing issues first.** If your bug is already reported, add a comment with your device + a "I can reproduce this" rather than opening a duplicate.
- **One bug per issue.** Don't bundle multiple problems into a single report.
- **Be specific.** "It doesn't work" is not actionable. "Tapping *Save* on the New Transaction screen on Android 14, Pixel 7, build 1.0.3 returns me to the list but the transaction is not added" is.

## ⚠️ Important: this repo is PUBLIC

Anything you post here — text, screenshots, logs, attachments — is **visible to the entire internet**. Before you submit, make sure your report contains:

- ❌ **No real financial data.** Use the seeded demo data or fake amounts. Never paste exported transactions, real balances, or anything from a real account.
- ❌ **No personal information.** No real names, emails, phone numbers, addresses, document numbers.
- ❌ **No tokens or credentials.** No API keys, session tokens, passwords. If a log line contains one, redact it (`Bearer xxx...`).
- ❌ **No screenshots of real banking apps or statements** in the background.
- ✅ Demo accounts, fake categories, screenshots of the test build only.

If you're not sure whether something is sensitive, leave it out and ask in the issue.

## What happens after you submit

- A maintainer triages the issue and adds it to the **Finance Tracker QA Project board**.
- You may be asked follow-up questions — please respond, otherwise the report may be closed as `needs-info`.
- Fixes happen in the private code repository. When a fix ships, the issue here is updated with the build version and closed.

## Build info to include

When reporting, please always note:

- **App version / build number** (visible in Settings → About, or on the build page where you got the APK).
- **Platform**: Android or iOS.
- **OS version**: e.g. Android 14, iOS 17.4.
- **Device model**: e.g. Pixel 7, iPhone 13, Samsung Galaxy A54.
- **Locale / language** the app was set to.

## Questions

For general questions or discussion (not bugs), use the [**Discussions**](https://github.com/ericocampos/finance-tracker-bugs/discussions) tab.

For first-time testers, please read the pinned **[Welcome QA testers](https://github.com/ericocampos/finance-tracker-bugs/discussions/1)** post before filing anything.
