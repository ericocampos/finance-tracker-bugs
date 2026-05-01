# Finance Tracker — How the app should behave

> **For QA testers.** This is the canonical reference for *what the app is supposed to do*. If the build you're testing does not match what's described here, that's a bug — file it.
>
> **Last updated:** 2026-04-29 (app v0.19.0).
>
> Maintainers update this file whenever app behavior changes. If something on screen contradicts this doc, **trust the doc** and report it.

---

## Table of contents

1. [What the app is](#1-what-the-app-is)
2. [Core concepts](#2-core-concepts)
3. [Validation rules and error messages](#3-validation-rules-and-error-messages)
4. [Workflows](#4-workflows)
5. [Screens](#5-screens)
6. [Balances and totals — how the numbers are computed](#6-balances-and-totals--how-the-numbers-are-computed)
7. [Things that should never happen (try them anyway)](#7-things-that-should-never-happen-try-them-anyway)
8. [Out of scope — please don't file these as bugs](#8-out-of-scope--please-dont-file-these-as-bugs)
9. [Glossary (EN / pt-PT / pt-BR)](#9-glossary-en--pt-pt--pt-br)

---

## 1. What the app is

A personal finance tracker for one user per device. The user records every movement of money — income they receive, expenses they pay, and transfers between their own accounts — and the app shows their balances and how the current month is going.

Key facts:

- **One user per device.** No multi-user, no households, no shared ledgers.
- **All data lives on the device.** The app does not send any financial data to any server. Backups go wherever the user picks in the OS share sheet (iCloud Drive, Google Drive, email, etc.); the app itself doesn't know where they end up.
- **Three languages**: English (`en`), European Portuguese (`pt-PT`), Brazilian Portuguese (`pt-BR`). The user chooses one **explicitly on the very first launch** via a blocking language picker (see §4.1) and can change it any time in Settings.
- **Three currencies for display**: EUR, USD, BRL. Currency only affects how amounts are formatted on screen — there's no currency conversion. The user picks one currency and lives in it.

---

## 2. Core concepts

### 2.1 Account

An "account" is a place the user keeps money — a checking account, a savings account, a wallet, a credit card, etc.

What the user can set:

- **Name** — must be unique among the user's active (non-archived) accounts.
- **Kind** — `Checking`, `Savings`, or `Other`. (The `Savings` kind unlocks the "opening balance" feature; see §4.5.)
- **Notes** — free-text, optional.
- **Order** — accounts can be reordered; the order is preserved across sessions.

Lifecycle: accounts can be **created**, **edited**, and **archived**. There is **no delete** — archived accounts are hidden from new-transaction pickers and from the Home account-balance strip, but their history stays visible in the Ledger.

### 2.2 Category

A category is a label attached to income or expense transactions (never to transfers).

What the user can set:

- **Name** — must be unique among the user's active categories.
- **Kind** — `Income` or `Expense`. This determines which transactions the category can be used on.
- **Color** — a swatch shown next to the category in lists.
- **Order** — same as accounts.

**On the very first launch**, after the user picks (or accepts) a language, the app seeds **19 default categories in that language**:

- 16 expense categories (e.g. Mercado, Alimentação, Transporte, Moradia, Saúde, Lazer, Educação, Roupas, Serviços, Presentes, Viagem, Outros, …).
- 4 income categories (e.g. Salário, Freelance, Investimentos, Outros).

**If the user later changes the language**, existing categories keep the names they were seeded with — they are user data once written. New categories created after the switch are written in the new language.

Same lifecycle as accounts: create, edit, archive. No delete.

### 2.3 Transaction

The core record. There are four types:

| Type | Direction | Carries a category? |
|---|---|---|
| **Income** | Adds money to one account | Yes (income category) |
| **Expense** | Removes money from one account | Yes (expense category) |
| **Transfer (out leg)** | Removes money from the *from* account | No |
| **Transfer (in leg)** | Adds money to the *to* account | No |

A **transfer** is always **two linked rows** (one out, one in) — you'll never see only one half of a transfer. Both halves share the same amount, date, description, and observations. If you edit one leg, both update; if you delete one leg, both delete.

What the user can set on a transaction:

- **Amount** — always shown as a positive number. The sign comes from the transaction type.
- **Date** — the date the money actually moved (not when the row was entered). Format: a calendar picker.
- **Description** — free text, the "where" or "who" (e.g. "Pingo Doce", "Salary"). Optional.
- **Observations** — free text, multi-line, the "why" or extra notes. Optional.

### 2.4 Settings

Stored per device, persists between launches:

- **Language** — `en`, `pt-PT`, or `pt-BR`.
- **Currency** — `EUR`, `USD`, or `BRL`.

---

## 3. Validation rules and error messages

When validation fails, the form stays open with the user's input intact and shows an inline red error under the field or at the bottom of the form. The exact strings below are what testers should see (they're translated to the active language).

### 3.1 All transaction types

| Rule | Error string (pt-BR / pt-PT) | Error string (EN) |
|---|---|---|
| Amount must be a positive number greater than zero | "Valor obrigatório" | "Amount must be positive" |
| Amount must not exceed €99,999,999.99 | "Valor demasiado alto. Verifique o número introduzido." (pt-PT) / "Valor muito alto. Verifique o número digitado." (pt-BR) | "Amount is too large. Please double-check the value." |
| Date must be a valid calendar date | "Data inválida" | "Invalid date" |

> **Tester note.** The amount field caps the digit count at 10 as you type, so you should not be able to enter values above €99,999,999.99. If the field accepts an 11th digit, that is itself a bug. The submit-time error is a defensive second layer; reaching it requires bypassing the digit cap somehow.

### 3.2 Income / Expense

| Rule | Error string (pt-BR / pt-PT) | Error string (EN) |
|---|---|---|
| Account must be selected | "Conta obrigatória" | "Account is required" |
| Category must be selected | "Categoria obrigatória" | "Category is required" |

### 3.3 Transfer

| Rule | Error string (pt-BR / pt-PT) | Error string (EN) |
|---|---|---|
| Both From and To accounts must be selected | "Conta obrigatória" | "Account is required" |
| From and To accounts must be different | "A conta de origem e destino devem ser diferentes" | "Source and destination must differ" |

> **Tester note.** The form *also* prevents the same-account case **before** validation by graying out the chip in the second picker that matches the first picker's selection. So the only way to hit the error string is to bypass that — which shouldn't be possible. **If you find a way, that's a bug worth filing.**

Transfers must **not** carry a category. The form hides the category picker entirely in transfer mode.

### 3.4 Opening balance

| Rule | Error string (pt-BR / pt-PT) | Error string (EN) |
|---|---|---|
| Only one opening balance per account per calendar month | "Já existe um saldo inicial para essa conta neste mês." | "An opening balance already exists for that account this month." |

The "Saldo inicial do mês / Opening balance for the month" toggle is **only visible** when both:

- The transaction type is **Income**, AND
- The selected account has kind **Savings**.

Editing an existing opening-balance row is fine — moving its date within the same month does not conflict with itself.

---

## 4. Workflows

### 4.1 First launch (fresh install or after restore on a new device)

The first launch shows up to **two onboarding screens** before the Home screen. Existing users who upgrade from a previous version (already have data) see neither — they go straight to Home.

#### Step 1 — Language picker (always shown on fresh install)

A full-screen, **blocking** screen with a stacked trilingual title:

> Choose your language
> Escolha o seu idioma
> Escolha seu idioma

Three rows, each showing a language name in its own language (the **endonym**):

- **English**
- **Português (Portugal)**
- **Português (Brasil)**

A **Continue / Continuar** button at the bottom. The button label is shown in the *highlighted* row's language.

**Pre-highlight rules** (the app reads the device's preferred language once at launch):

- Device language is **Português (Brasil)** → "Português (Brasil)" row pre-highlighted, Continue reads **Continuar**, button enabled.
- Device language is **Português (Portugal)** → "Português (Portugal)" row pre-highlighted, Continue reads **Continuar**, button enabled.
- Device language is **English (any region)** → "English" row pre-highlighted, Continue reads **Continue**, button enabled.
- Device language is anything else (e.g. Spanish, French) → **no row pre-highlighted**, Continue button is **disabled** until the user taps a row.

The user **must tap one of the rows and then tap Continue**. There is no Skip and no Back.

> **Tester note (language picker).**
> - Verify that on a freshly installed app, the picker really appears, with no other UI shown first.
> - Verify the pre-highlight matches the device language.
> - Verify the Continue button is disabled until any row is tapped (only relevant when no row was pre-highlighted, i.e. an unsupported device locale).
> - Verify that tapping Continue takes you to the next step (or to Home if biometric onboarding is skipped per the rules below).
> - Verify that **closing and re-opening the app** does **not** show the picker again — the choice is remembered.

#### Step 2 — Biometric / passcode lock onboarding (sometimes shown)

This screen appears immediately after the user taps Continue on the language picker, **only** if the device has biometric (Face ID / fingerprint) **or** a device PIN/passcode set up. If the device has neither, this screen is silently skipped.

A full-screen prompt with:

- **Title** (in the chosen language): "Protect your data" / "Proteja os seus dados" / "Proteja seus dados".
- A short body explaining that the app holds financial data and that the device's existing biometric or PIN can be used to lock it, and that this can be changed later in Settings.
- A primary button: **Enable / Ativar**.
- A secondary text-style button: **Skip for now / Agora não**.

**Tap "Enable":**
- The OS biometric/passcode prompt appears with the message **"Confirm to enable app lock"** (localized).
- On **success**, the app proceeds to Home with the lock turned on.
- On **cancel** or **fail**, the onboarding screen **stays visible** so the user can try Enable again or tap Skip.

**Tap "Skip for now":** the lock stays off and the app proceeds to Home. The user can enable it later in **Settings → Segurança / Security**.

> **Tester note (biometric onboarding).**
> - On a device with biometric or passcode enrolled: verify the screen appears immediately after Continue on the language picker.
> - On a device with **no** biometric and no PIN at all: verify the screen does **not** appear; the app jumps from the language picker straight to Home.
> - Verify "Enable → cancel OS prompt" leaves you on the onboarding screen (not on Home) and the OS prompt can be triggered again.
> - Verify "Enable → succeed" → force-quit → relaunch shows the **App bloqueada** lock screen (cold-start lock works).
> - Verify "Skip" → relaunch does **not** show the lock screen.
> - Verify that on subsequent launches (after either Enable or Skip), this screen never reappears unless app data is wiped.

#### Step 3 — Home screen and default categories

1. The app opens to the Home screen with **zero accounts**.
2. The user is expected to go to **Settings → Contas / Accounts → +** to create at least one account before adding any transactions.
3. The 19 default categories are pre-seeded in the language picked above; the user can use them immediately, edit them, or archive them.

> **Tester note (categories).** Verify the seeded category names match the language picked on Step 1. Verify that *changing the language afterward* does **not** rename existing categories.

### 4.2 Adding a transaction

1. From **Home → tap the floating "+" button (FAB)**, or from the **Ledger → tap "+"**, to open the transaction modal.
2. Pick **Receita / Income**, **Despesa / Expense**, or **Transferência / Transfer** at the top.
3. Fill the fields:
   - **Income or Expense**: Account → Category (only categories of the matching kind appear) → Amount → Date → Description → Observations.
   - **Transfer**: From account → To account (the chip matching the From selection is grayed out) → Amount → Date → Description → Observations. **No category picker is shown.**
   - **Income on a Savings account**: a "Saldo inicial do mês / Opening balance for the month" toggle appears below the category picker.
4. Tap **Save**.
5. On success the modal closes and the user goes back to the screen they came from. The new transaction is immediately reflected in totals and balances.
6. On validation failure the modal stays open, the user's values are preserved, and the relevant error is shown.

### 4.3 Editing a transaction

1. From the **Ledger**, tap any row to open the modal in edit mode.
2. Every field is prefilled. **The transaction type cannot be changed** (an income can never become an expense; a transfer can never become a single-sided row).
3. For a **transfer**, the modal automatically loads the paired leg and prefills both From and To.
4. On Save:
   - **Income / Expense** → updates the row.
   - **Transfer** → updates **both legs** at the same time. Shared fields (amount, date, description, observations) propagate to both. Account changes apply per side, so the user can swap From and To, or change either side independently — the pair always stays consistent.

### 4.4 Deleting a transaction

1. From the edit modal, tap the destructive **"Apagar / Delete"** button at the bottom.
2. Effect:
   - **Income / Expense** → removes the row.
   - **Transfer** (either leg) → removes **both** legs.
3. The modal closes. **There is no undo and no soft-delete** — the row is gone immediately.

### 4.5 Opening balance — what it is and why

Savings accounts in the user's mental model carry over from month to month, but the app only stores the transactions the user actually enters. So once a month, on each savings account, the user records an "opening balance" income row representing whatever the bank statement says the account is at on day one.

Behavior:

- **How to create one**: New transaction → Income → pick a Savings account → toggle "Saldo inicial" on → Save.
- **It does NOT count as monthly income.** Opening-balance rows are **excluded** from the monthly Income / Expense / Net cards on Home, and from monthly aggregates in general.
- **It DOES count toward the running balance.** See §6.2 for how.
- **Visual marker**: the Ledger row shows a small "saldo inicial" chip next to the description, so the user can spot them.
- **One per account per month** (see §3.4). Editing the date within the same month is fine.
- **Toggling the flag off** on an existing row converts it back to a normal income row.

### 4.6 Account management

**Settings → Contas / Accounts**. Standard list with a `+` button. Tap a row to edit. The edit screen exposes name, kind, notes, and an **Archive** toggle. Archived accounts are hidden from the active list by default — a filter toggle reveals them.

### 4.7 Category management

**Settings → Categorias / Categories**. Same pattern. Edit screen exposes name, kind, color picker, and Archive toggle.

### 4.8 Changing language and currency

Two chip pickers at the top of Settings. Tapping a chip:

- **Persists** the choice immediately.
- For language: re-renders **all UI strings** in the new language right away (no app restart needed).
- For currency: re-renders **all amounts** in the new currency format right away.

Existing user data (account names, category names, transaction descriptions, observations) is **never auto-translated**.

### 4.9 Backup — export

**Settings → Backup → "Exportar backup"**.

1. The app generates a backup file with a timestamped filename like `finance-tracker-2026-04-26-1430.db`.
2. The OS share sheet opens — the user picks where the file goes (iCloud Drive, Google Drive, email, AirDrop, save to Files, anywhere).
3. The app does **not** track where the backup ended up. The user is responsible for keeping it somewhere safe.

> **Tester note.** Backup file rotation, encryption, and storage are deliberately **not** done by the app. Don't file "the app should encrypt backups" as a bug — that's an explicit non-feature (see §8).

### 4.10 Backup — restore

**Settings → Backup → "Restaurar backup"**.

1. A destructive **confirmation alert** appears: *"Esta ação substitui todas as transações, contas e categorias atuais. Continuar?"* — Cancel / Restore. Tapping outside should **not** confirm.
2. On Restore, a **file picker** opens, scoped to SQLite database files.
3. After the user picks a file, the app replaces the local database with the contents of the picked file and shows a full-screen message: *"Backup restaurado / Backup restored — force-quit and reopen"*. The bottom tabs become unreachable.
4. The user **must force-quit the app and reopen it.**
5. On the next cold start the app boots normally with the restored data.

Edge cases worth probing:

- **Restoring an older backup.** Should work — any internal upgrades apply on the next boot.
- **Restoring a file that isn't a real database.** The OS file picker filters by file type; if a tester somehow selects a non-database file (a renamed `.db`, etc.), the app should fail gracefully on next boot rather than corrupting silently.
- **Killing the app between confirming the restore and seeing the force-quit message.** On next launch the user should see the restored data with no error.

### 4.11 Theme — light, dark, follow the system

**Settings → Tema / Theme**. Three chip options:

- **Sistema / System** *(default)* — follows the device's color scheme **live**. If the user flips the OS theme while the app is open, every screen re-renders without needing a relaunch.
- **Claro / Light** — pins the app to light mode regardless of the OS setting.
- **Escuro / Dark** — pins the app to dark mode regardless of the OS setting.

The choice is persisted across app launches.

> **Tester note.** Every screen — Home, Ledger, Insights, Settings, the new-transaction and edit modals, the lock screen, and the post-restore "force-quit" screen — must have a dark-mode treatment. **No screen should ever show bright-white text on bright-white background, illegible contrast, or "white flash" on transition.** Active accent buttons and chips invert in dark mode (dark bg in light mode → light bg in dark mode) so they stay visually prominent. The status bar tint should match the active theme automatically.

### 4.12 Biometric lock

Optional security gate. **Settings → Segurança / Security → "Trancar com biometria" toggle**. Default off.

- **First-launch onboarding.** On a fresh install with biometric/passcode enrolled, the user is asked to enable this feature during onboarding (see §4.1, Step 2). Skipping there leaves the toggle off and the user can flip it on later from Settings using the rules below.
- **Availability check.** If the device has no enrolled biometrics (Face ID, fingerprint) AND no device PIN, the toggle is **disabled** with a caption: *"Configure biometria nas definições do dispositivo."* Tapping the disabled toggle does nothing.
- **Toggling the switch (NEW in v0.19.0).** Tapping the toggle — **in either direction (off → on AND on → off)** — immediately fires the OS biometric/passcode prompt with the message **"Confirm to change app lock"** (localized). The toggle's visible position and the lock state are only updated after a successful authentication.
  - **OS prompt cancelled or failed:** the toggle visibly stays where it was. Nothing else changes. The user can tap it again to retry.
  - **OS prompt succeeded:** the toggle flips, and the lock state changes accordingly.
- **Cold start (toggle on).** When the user opens the app, a full-screen **"App bloqueada"** lock screen appears *before* the tabs render. The OS biometric prompt fires automatically. On success, the app appears.
- **Background return (toggle on).** Returning to the app from the background **after more than 60 seconds** re-locks it. Returning sooner does **not** re-lock. iOS "inactive" state (e.g. when a system Alert is showing on top of the app) does **not** trigger the lock — only true backgrounding.
- **Auth failure on the lock screen.** The lock screen stays put; the user can tap **"Desbloquear"** to retry. The OS prompt offers a passcode fallback if biometrics keep failing.

> **Tester note (toggle re-auth, NEW in v0.19.0).**
> - Verify that with the toggle currently OFF, tapping it triggers the OS prompt. Cancel → toggle stays OFF. Succeed → toggle goes ON.
> - Verify that with the toggle currently ON, tapping it triggers the OS prompt. Cancel → toggle stays ON. Succeed → toggle goes OFF.
> - Verify that on a device with no biometric/PIN enrolled, the toggle is greyed out and tapping it does nothing.

### 4.13 Geo-tagging — capturing where a transaction happened

Transactions can optionally carry a captured location (latitude, longitude, accuracy in metres). Geo capture is **purely opt-in per transaction** — the default is no location.

- **Capture button.** Inside the new-transaction modal *and* the edit modal, below the Observations field, a **"📍 Capturar localização"** button.
- **First tap → OS permission prompt.** Subsequent taps capture silently.
- **Captured state.** The button area turns into a card showing **"Localização guardada (≈Xm)"** with **"Recapturar"** and **"Limpar"** links.
- **Permission denied.** Button stays visible with caption *"Permissão negada nas definições do dispositivo."* The user can re-enable in OS Settings and try again.
- **Location services off OS-wide.** Caption *"Serviços de localização desativados."*
- **Transfers.** Capturing on a transfer writes the same coordinates to **both** legs. Recapturing or clearing on the edit modal cascades to the paired leg, same as the amount/date/description cascade.
- **Display in the Ledger.** Rows that have a captured location render a small **map-pin icon** next to the description.
- **Storage / privacy.** Coordinates **stay on the device**. They are included in the SQLite backup file the user exports manually; they are never sent to any server.

> **Tester note.** Coordinates leaving the device — under any circumstance — would be a critical bug. If you see network activity correlated with capturing a location, file it as a Crash/security issue immediately and stop testing.

---

## 5. Screens

The app has **four bottom tabs**: **Início / Home**, **Lançamentos / Ledger**, **Insights / Insights**, **Configurações / Settings**.

> **Tab bar accessibility.** The active tab is distinguished from inactive tabs by **geometry** (e.g. filled inner dot, taller bar in Insights), not just color. This is intentional — the active tab must remain identifiable in grayscale screenshots and for color-blind users. **A bug worth filing**: an active tab that's only distinguishable by color tint, not by shape.

### 5.1 Home — month at a glance

- **Header**: current month name and year.
- **Three summary cards**: total Income, total Expense, and Net (Income − Expense) for the **current calendar month**. Opening-balance rows are **excluded** from these numbers.
- **Per-account balance strip**: a horizontal scroll showing the running balance of every non-archived account, in the user's chosen order.
- **Floating "+" button** → opens the new-transaction modal.

### 5.2 Ledger — browse, filter, edit

- **Month navigator**: `< [Month] [Year] >` at the top. Tap the arrows to move month-by-month.
- **Filter chips**: **Todas / All**, **Receita / Income**, **Despesa / Expense**, **Transferência / Transfer**. Multiple chips can be active at once; the active set is the filter.
- **Day-grouped list**: a header for each day with that day's net total, then the rows for the day. Each row shows:
  - The category-color swatch.
  - The description (or category name, or account name, falling back to "—" if all empty).
  - A sub-line: **"account • category"**.
  - The **signed amount** (positive in green for income / transfer-in, negative in red for expense / transfer-out).
- Opening-balance rows render the **"saldo inicial"** chip next to the description.
- Rows that have a captured location render a small **map-pin icon** next to the description (see §4.13).
- Tap any row → edit modal.
- **Map icon in the header** → opens the modal Map view for the displayed month (see §5.5).

**Empty states (two distinct cases — please don't confuse them when filing bugs):**

- **The displayed month has zero transactions at all** (before any chip filter is applied) → the list renders a **branded empty state**: circle/ring artwork, the title *"Nada por aqui ainda"*, a privacy-leaning subtitle, and a primary CTA *"Novo lançamento"* that opens the new-transaction modal.
- **The month has transactions but the active filter chips exclude all of them** → a compact text fallback *"Sem transações neste mês."* (no artwork, no CTA).

### 5.3 Insights — analytics over the last 6 months

The fourth bottom tab. Shows patterns in the data the user has entered. The window is the **rolling last 6 months ending at the current calendar month** — there is **no time-range scrolling** in this version. All values aggregate across **all non-archived accounts**; there is **no per-account filter**.

The screen has four sections, top to bottom, each in its own card:

- **Mensal / Monthly** — grouped bar chart, one bar pair per month: green income, red expense. A small label above each pair shows that month's net. Opening-balance income rows are **excluded**; transfer legs are **excluded**.
- **Saldo acumulado / Cumulative balance** — line chart, one point per end-of-month for the 6 months. Each point is the **aggregate balance of all accounts** at that cutoff, computed using the same anchor semantics that drive Home's per-account balances (most recent opening-balance row on or before the cutoff anchors that account's running total; signed transactions then accumulate).
- **Despesas por categoria / Expenses by category** AND **Receitas por categoria / Income by category** — two donut charts (current month only) plus a legend. Each legend row shows: color swatch · category name · percent · amount, sorted **descending by amount**. Categories with zero in the current month don't appear. If a kind has no rows that month, a small grey caption (*"Sem despesas este mês." / "No expenses this month."*) replaces the chart for that section. Transfer legs and opening-balance rows are **excluded**.
- **Maiores variações / Biggest movers** — list (no chart). Two sub-sections:
  - **Top 3 expense categories** ranked by **absolute change vs. last month** (sign shown via ↑/↓ chip).
  - **Top 3 income categories** by **current-month total** (no chip).
  - Ties broken by the larger of (current, previous) totals. Categories that exist in the prior month but not in the current still appear if their delta puts them in the top 3. Opening-balance rows are excluded.

**Empty / sparse data behaviour:**

- **Zero transactions in the 6-month window** → the entire screen shows a single empty state with a CTA jumping to the new-transaction modal.
- **Only the current month has data** → trend bars and cumulative line show one populated point; a small grey caption below the page reads *"Adicione mais meses para ver tendências."*
- **One kind missing in the breakdown** → that kind's card shows the placeholder caption; the other kind renders normally.
- **Fewer than 3 movers in either sub-section** → render whatever exists; if zero, the entire "Biggest movers" card is hidden.
- **Out-of-scale data behaviour.** If any monthly aggregate or any cumulative-balance point in the 6-month window exceeds **€100M** (or the equivalent in the active currency), the affected card replaces its bar/line chart with a textual fallback: a short caption (*"Valores fora de escala. Verifique a sua lista de lançamentos." / "Values are off the chart. Please double-check your transactions."*) followed by the per-month numbers in a list. The Despesas/Receitas donut and the Maiores variações list are **not** affected — they format with `formatMoney` and remain readable at any value. This is a defensive guard against pre-existing outlier rows; the input cap in §3.1 prevents new entries from triggering it.

**Refresh behaviour**: the screen **refetches on every focus**. So adding a transaction in another tab and returning to Insights should immediately reflect the new data — no manual refresh control exists.

> **Tester note.** Insights is heavily dependent on the same computation rules as Home (§6). If a number on Insights disagrees with what you can recompute from the Ledger, that's a high-value bug — please file with the exact values you saw and the source rows.
>
> If the Mensal or Saldo acumulado card shows the textual "Valores fora de escala" fallback unexpectedly (i.e. without an obviously-large transaction in your data), that's also worth reporting — please include the per-month figures shown.

### 5.4 Map — geo-tagged transactions for the displayed month

A **modal map screen** opened from the **map icon in the Ledger header**. Shows every transaction in the **same month the Ledger is currently displaying** that has a captured location.

- **Markers**: one pin per transaction. Pin colour by transaction type — **green** for income, **red** for expense, **grey** for transfers. Multiple transactions at the same coordinate are allowed and stack visually.
- **Camera on open**: fits the bounding box of all pins with edge padding. If the month has no geo-tagged rows, falls back to a wide-zoom default view and shows the empty-state overlay.
- **Tapping a marker**: opens a callout — description (or category fallback) on top, **signed amount** in the middle, **occurrence date** below. Tapping the callout itself opens the transaction's edit modal.
- **Empty state**: a semi-transparent caption *"Sem transações com localização neste mês."* over the map.
- **Header**: a close button (top-right) and a title showing the month name. Status bar inset is respected.
- **Privacy**: the map renders entirely from local data. **No coordinate data leaves the device.**

> **Tester note.** This screen requires a properly built APK (Google Maps API key baked in at build time). **Expo Go cannot render this screen.** Always test the Map screen using the APK pinned in the README's "Current test build" — not Expo Go. If you see "map area is grey/blank" on the pinned APK build, that's a bug worth filing; on Expo Go it is expected.

### 5.5 Settings — preferences and data

- **Top**: Language picker (chips), Currency picker (chips).
- **Theme picker** (chips: System / Light / Dark) — see §4.11.
- **Sub-routes**: Contas / Accounts, Categorias / Categories.
- **Segurança / Security card**: Biometric lock toggle — see §4.12.
- **Backup card**: Export, Restore.
- **Sobre / About card** (bottom of the page): displays the running app version (e.g. `Versão 0.19.1`). Always include this value when you file a bug report — paste it into the *App version / build number* field on the issue template.

---

## 6. Balances and totals — how the numbers are computed

> Use this section to verify the numbers shown on screen match what they should be. Bugs in computed totals are some of the highest-impact reports you can file.

### 6.1 Monthly totals (Home cards, Ledger day headers)

For the displayed month and (if filtered) account scope:

- **Income** = sum of all **Income** transactions, **excluding** opening-balance rows.
- **Expense** = sum of all **Expense** transactions.
- **Net** = Income − Expense.
- **Transfer legs are excluded** from all three numbers (transfers move money between the user's own accounts; they're not income or expense).

### 6.2 Running balance per account (Home strip)

Computed by walking the account's transactions in order of date, then by entry order on the same date:

- **If the account has at least one opening-balance row** dated on or before "today":
  - The balance starts from the **most recent** opening-balance row.
  - Then **every transaction strictly after** that row is added with its sign.
- **If the account has no opening-balance row**:
  - The balance is simply the signed sum of every transaction up to today.
- **Sign rules**:
  - **Income** and **Transfer-in** → add the amount.
  - **Expense** and **Transfer-out** → subtract the amount.

### Try this — common balance scenarios to verify

- A savings account with one opening-balance row in this month and several income/expense rows after it: balance should = opening balance + (later income) − (later expense).
- A checking account with no opening balances: balance should = sum of all income + transfers-in − all expenses − transfers-out, ever.
- A transfer of 100 from Account A to Account B: A's balance goes down by 100, B's balance goes up by 100, and the **monthly Income / Expense / Net cards on Home don't change**.
- Editing an opening-balance row's amount should immediately update the running balance.
- Archiving an account should remove it from the Home strip but keep its rows visible in the Ledger.

---

## 7. Things that should never happen (try them anyway)

These are states the app is supposed to prevent. **If you can produce any of them, that's a high-severity bug.**

- **Two opening-balance rows on the same account in the same calendar month.** The app should reject the second one with the error in §3.4.
- **A transfer where From and To are the same account.** The form grays out the chip; if you can submit it anyway, that's a bug.
- **A transfer leg that has a category.** Transfers should never carry a category.
- **A category whose kind doesn't match the transaction type** (e.g. picking an Expense category on an Income transaction). The form filters the picker to prevent this; if you can produce a mismatch, file it.
- **A transfer where editing one leg leaves the other leg out of sync** (different amount / date / description). The pair should always stay consistent.
- **Deleting one leg of a transfer leaves the other leg orphaned.** Both legs should disappear together.
- **Submitting the form before accounts have loaded.** The Account picker should be empty / disabled until data is ready.
- **Confirming a restore by tapping outside the alert** rather than the explicit "Restore" button.
- **A captured location persisting on a transfer leg without the paired leg also having it.** Capturing, recapturing, or clearing a location on one leg of a transfer must cascade to the other leg.
- **Coordinates leaving the device.** If you observe network activity correlated with capturing or saving a location, file it as a critical security bug immediately.
- **The Map screen showing pins from a different month** than what the Ledger is currently displaying. The two should always be in sync.
- **The biometric lock failing to re-engage after the user backgrounds the app for more than 60 seconds** (with the toggle on). Or, conversely, locking under 60 seconds.
- **The biometric lock toggle being enabled on a device with no enrolled biometrics or PIN.** It should be greyed out.
- **A theme that doesn't apply to a screen** — e.g. dark mode active everywhere except one modal that stays light. List the specific screen + action that produced it.
- **Insights numbers disagreeing with Ledger.** If the Insights monthly bar for a given month shows a different income/expense total than what you can verify by summing the Ledger rows for that month (excluding opening-balance rows and transfers), that's a high-value bug. Include both numbers and the source rows.

---

## 8. Out of scope — please don't file these as bugs

The following are **deliberate non-features** in the current build. Filing these as bugs adds noise. If you have feedback about whether they *should* exist, open a **Discussion** instead.

- **Investments** (holdings, snapshot prices, performance) — planned but not in mobile yet.
- **Per-account filter on Insights** — Insights aggregates across **all non-archived accounts**; there's no way to scope a chart to a single account in this version.
- **Time-range scrolling on Insights** — the window is locked to the rolling last 6 months ending at the current calendar month.
- **AI / LLM analysis or narratives inside the app** — explicitly out of scope by design (the app must not send financial data anywhere).
- **Sync between two devices, multi-user accounts, household sharing.**
- **Automatic / scheduled / periodic backups, backup rotation, backup encryption.** Backup is a manual user action by design.
- **CSV or JSON import/export.**
- **Recurring transactions, scheduled future transactions, transaction templates.** The app accepts any date the user types but does not project anything forward.
- **Currency conversion.** The user picks one currency; amounts are not converted.
- **Hard-deleting an account or a category.** Only Archive is exposed.
- **App store store-listing metadata, version naming policy, store submission UX.** These are release-pipeline concerns, not in-app behaviour — please don't file them as app bugs.
- **Map screen on Expo Go.** The Map screen requires a built APK with a Google Maps API key — Expo Go renders it grey/blank and that's expected (see §5.4).

---

## 9. Glossary (EN / pt-PT / pt-BR)

These are the canonical user-facing terms. If the app shows something different, it's likely a translation bug — please file under the **Translation / i18n** template.

| English | Português (Portugal) | Português (Brasil) |
|---|---|---|
| Account | Conta | Conta |
| Category | Categoria | Categoria |
| Income | Receita | Receita |
| Expense | Despesa | Despesa |
| Transfer | Transferência | Transferência |
| Opening balance | Saldo inicial | Saldo inicial |
| Description (a.k.a. merchant) | Descrição | Descrição |
| Observations | Observações | Observações |
| Backup | Cópia de segurança | Backup |
| Restore | Restaurar | Restaurar |
| Settings | Definições | Configurações |
| Home | Início | Início |
| Ledger | Lançamentos | Lançamentos |

---

## Found something this doc doesn't cover?

If the app does something not described here, that's interesting — it's either:

- **A bug** (the doc is right, the app is wrong) → file an Issue.
- **A doc gap** (the app is right, the doc is incomplete) → open a Discussion so a maintainer can update this file.

Either way, don't sit on it. Speak up.
