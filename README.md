# Finance Tracker — QA & Bug Reports

This repository is the **public bug tracker** for the Finance Tracker mobile app. The app source code lives in a separate private repository; this repo contains **no code** — only issue templates and reporting guidelines.

If you are a QA tester, this is the right place. Welcome, and thank you for testing.

## 📦 Current test build

> **Test only the build linked here.** Reports from older builds, side-loaded variants, or unofficial copies cannot be triaged.

| Field | Value |
|---|---|
| **Platform** | Android (APK + OTA) |
| **App version** | `0.19.2` (delivered via OTA on top of the 0.19.1 APK below) |
| **Install link (one-time, APK)** | https://expo.dev/accounts/ericocampos/projects/finance-tracker-mobile/builds/4441ff84-64da-4377-a22c-123bded6de40 |
| **Direct APK download** | https://expo.dev/artifacts/eas/9qSn9SSF92A6TCaV7To9L9.apk |
| **APK Build ID** | `4441ff84-64da-4377-a22c-123bded6de40` (the 0.19.1 build; this is the current OTA runtime base) |
| **OTA tag (current)** | `mobile-v0.19.2-preview1` — published 2026-05-02. The OTA arrives automatically on next foreground; **no reinstall needed** if you're already on 0.19.1+. |
| **How to confirm you're on the latest** | Open the app → Configurações → Sobre → check the version. Should read `0.19.2`. If you still see `0.19.1` after a foreground/background cycle and a network connection, force-quit and reopen. |
| **OTA channel** | `preview`, matching `runtimeVersion: "1"` (hand-managed, decoupled from app version). Future patch releases (0.19.3, 0.19.4, …) on the same runtime arrive as silent OTAs — no APK reinstall. A fresh APK is only required when the runtime version itself changes (native module additions or Expo SDK upgrade). |
| **What's new in 0.19.2 (this OTA)** | **Per-bar Insights fallback** — when only some months in the 6-month window have an unusually large value, the chart now renders the in-range months normally and only the outlier slot gets a dashed-ghost marker. Previously the entire card collapsed to a textual list. A caption row below the chart names the outlier month and shows its actual value. See `docs/app-behavior.md` §5.3 for the updated behaviour. |
| **What's in the 0.19.1 base APK** | **Insights freeze fix** — tapping the Insights tab no longer soft-locks the app on devices that have a transaction with an unusually large amount. **Amount cap** — the new-transaction amount field is capped at €99,999,999.99 (10 digits max) to prevent zero-typo crashes. **Settings → Sobre** — displays the running app version. See `docs/app-behavior.md` §3.1 and §5.5. |

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
