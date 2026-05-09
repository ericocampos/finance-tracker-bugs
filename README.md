# Finance Tracker — QA & Bug Reports

This repository is the **public bug tracker** for the Finance Tracker mobile app. The app source code lives in a separate private repository; this repo contains **no code** — only issue templates and reporting guidelines.

If you are a QA tester, this is the right place. Welcome, and thank you for testing.

## 📦 Current test build

> **Test only the build linked here.** Reports from older builds, side-loaded variants, or unofficial copies cannot be triaged.

| Field | Value |
|---|---|
| **Platform** | Android (APK + OTA) |
| **App version** | `0.19.18` (delivered via OTA on top of the 0.19.4 APK below) |
| **Install link (one-time, APK)** | https://expo.dev/accounts/ericocampos/projects/finance-tracker-mobile/builds/023b15db-8253-425e-ac18-6a9d59d7996e |
| **Direct APK download** | https://expo.dev/artifacts/eas/gYFixNbX3gSD6nbdu9eLA.apk |
| **APK Build ID** | `023b15db-8253-425e-ac18-6a9d59d7996e` (the 0.19.4 build; this is the current OTA runtime base) |
| **OTA tag (current)** | `mobile-v0.19.18-ota` — published 2026-05-09. The OTA arrives automatically on next foreground; **no reinstall needed** if you're already on 0.19.4 or any later 0.19.x OTA. The app shows an in-app prompt when a new OTA has been fetched — see *What's new* below. |
| **How to confirm you're on the latest** | Open the app → Configurações → Sobre → check the version. Should read `0.19.18`. If you still see an earlier `0.19.x` after a foreground/background cycle and a network connection, force-quit and reopen. You can also tap the **Versão** row directly to trigger a manual check. |
| **OTA channel** | `production`, mapped to `runtimeVersion: "1"` (hand-managed, decoupled from app version). Future patch releases on the same runtime arrive as silent OTAs — no APK reinstall needed. A fresh APK is only required when the runtime version itself changes (native module additions or Expo SDK upgrade). |
| **What's new since 0.19.8 (this OTA chain)** | **0.19.18 — Merchant → category suggestion.** When you leave the merchant input on a new- or edit-transaction form (in expense or income mode), the app checks whether the same merchant text has appeared on a previous transaction of the same type with a category set. If yes, a small chip "Sugerido: <category>" appears below the merchant input. Tapping the chip fills the category. The chip never overrides a category you've already chosen, never appears in transfer mode, and disappears as soon as you edit the merchant text, pick a category manually, or switch modes. Matching ignores case and accents (so "Padão" and "Padao" match each other). **0.19.17 — Versioned local snapshots.** The app now keeps a rolling history of automatic on-device backups so you can roll back to a known-good state if something goes wrong. Snapshots are taken automatically when the app comes to the foreground, capped at one every 24 hours, and stored entirely on the device (never uploaded). Retention: the most recent 7 daily snapshots plus 12 monthly anchors (the oldest snapshot of each prior month). A pre-restore snapshot is taken automatically before any restore (from snapshot or from external file) so you can always undo. New Settings → Backup → "Snapshots" sub-screen lists every snapshot with type (Auto / Pre-restore), relative timestamp ("today, 14:32" / "yesterday, 09:15" / "3 days ago, …"), size, and per-row Restore action; "Wipe all" clears the history. Pre-restore snapshots are protected from automatic deletion. See `docs/app-behavior.md` §2.7, §4.17, §5.7. **0.19.16 — Recurring-charge detector.** The app now detects monthly-recurring expenses (subscriptions, fixed bills) automatically. New "Gasto fixo mensal" / "Monthly fixed spending" card at the top of the Insights tab shows the sum of medians and a count of detected items. Tapping the card opens a new `/recurring` detail screen with a filter (Todos / Confirmados / Dispensados) and per-row Confirmar / Dispensar / Limpar actions. Detection runs over the last 6 months, requires ≥ 3 occurrences, ±20% amount tolerance. Without 3 months of expense history, the card shows an empty-state hint. See `docs/app-behavior.md` §2.5 (Pattern), §4.16, §5.6. **0.19.15 — Deep-link prefill on `transaction/new`.** The custom-scheme URL `finance-tracker://transaction/new?...` accepts query parameters (`amount`, `category`, `account`, `merchant`, `observations`) and pre-fills the new-transaction form. Unknown or invalid params are silently dropped; the form opens with whatever resolved cleanly. A new "Conta padrão para criação rápida" picker appears in Settings only when ≥ 2 active accounts exist. See `docs/app-behavior.md` §2.4 and §4.15. **0.19.14 — Personal tags.** Free-form, multi-valued labels per transaction (including transfer legs). New tag input below Categoria on the transaction form (type-ahead with inline create). New Settings → Tags screen for rename / archive / unarchive. New Insights "Por tag" breakdown. New tag filter chips on Ledger and Map. Tag colour is hash-derived (no manual picker). See `docs/app-behavior.md` §2.5 and §4.14. **0.19.13 — Status-bar safe-area fix.** The top status-bar gap is now reserved at the screen container, not inside scrolling content; scrollable content no longer travels behind the status bar. **0.19.12 — Currency symbol fix.** The selected currency symbol is now correctly shown in the transaction amount input (previously always showed €). **0.19.11 — Default account on first run + dark-mode empty state.** Fresh installs now seed a "Conta principal / Main account" so the user can log a transaction immediately on first launch. Dark-mode visual treatment fixes for the Ledger empty state. **0.19.10 — Ledger FAB + tap-to-jump month picker.** Floating "+" button anchored to the bottom-right of Ledger. Tapping the month-name header opens a quick year/month picker so you can jump directly to any month without scrolling. **0.19.9 — Home dashboard redesign.** The Home tab is now a vertically scrolling dashboard (greeting header, "this month" card, account-balance carousel, recent activity) instead of the flat list of cards. **0.19.8 — Manual "tap to check" on the Versão row.** When no update is pending, the Sobre / Versão row reads `Versão 0.19.X — toque para verificar` (pt) / `tap to check` (en); tapping fires an OTA check. While the check runs, the row briefly shows `a verificar… / verificando… / checking…` (disabled). If no update is found, it briefly reads `you're on the latest` for ~3 s. If the check fails (offline, network error), it briefly reads `couldn't check` for ~3 s in red and remains tappable for retry. **0.19.7 — In-app OTA update prompt.** When an OTA update has been downloaded and is waiting to be applied, the bottom Settings tab shows a small accent-coloured notification dot, and the Versão row flips to `… toque para reiniciar / tap to restart`. Tapping the row reloads the app on the new bundle. The auto-check fires on cold start and on app foreground, debounced to once every 30 minutes. **0.19.6 — Insights cumulative-balance fix.** The cumulative-balance line chart on the Insights tab no longer overflows below the card when balances go negative — the y-axis now mirrors symmetrically across zero so the whole line is visible without scrolling. (Closes public bug #7.) See `docs/app-behavior.md` §4.13 and §5.5. |
| **What's in the 0.19.4 base APK** | (1) **Build/OTA fix** — APK now embeds the `preview` EAS Update channel, so future JS-only fixes actually reach you. The earlier 0.19.1 APK was built without a channel and never received any OTA. (2) **i18n leak fixes** — Home tab labels (eyebrow / Income / Expenses / Net / Accounts), bottom-nav titles (Home / Ledger), Save & Delete buttons on the new/edit transaction screens, the Home date label, plus Settings sibling leaks (Add/Edit account & category screens, archive/unarchive labels, validation errors) all now respect the language setting. (3) **Insights outlier fallbacks** — per-bar / per-point dashed-ghost markers when only some months have an unusually large value, plus the original textual-list fallback for the all-large case. (4) **Insights freeze fix** and the €99,999,999.99 amount cap. **Default category names are NOT affected** — they are seeded into the on-device database at first launch in the language you picked then, and stay that way (changing the language afterwards re-translates UI strings, not stored data). See `docs/app-behavior.md` §3.1, §3.4, §5.3, §5.5. |

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
