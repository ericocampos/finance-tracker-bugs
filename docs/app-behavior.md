# Finance Tracker — How the app should behave

> **For QA testers.** This is the canonical reference for *what the app is supposed to do*. If the build you're testing does not match what's described here, that's a bug — file it.
>
> **Last updated:** 2026-04-27 (app v0.17.0).
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
- **Three languages**: English (`en`), European Portuguese (`pt-PT`), Brazilian Portuguese (`pt-BR`). The user chooses one in Settings. **First launch defaults to `pt-BR`.**
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
| Date must be a valid calendar date | "Data inválida" | "Invalid date" |

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

1. The app opens to the Home screen with **zero accounts**.
2. The user is expected to go to **Settings → Contas / Accounts → +** to create at least one account before adding any transactions.
3. The 19 default categories are pre-seeded in the active language; the user can use them immediately, edit them, or archive them.

> **Tester note.** Verify the seeded category names match the language that was active at first launch. Verify that *changing the language afterward* does **not** rename existing categories.

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

---

## 5. Screens

The app has **three bottom tabs**: **Início / Home**, **Lançamentos / Ledger**, **Configurações / Settings**.

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
- Tap any row → edit modal.

### 5.3 Settings — preferences and data

- **Top**: Language picker (chips), Currency picker (chips).
- **Sub-routes**: Contas / Accounts, Categorias / Categories.
- **Backup card**: Export, Restore.

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

---

## 8. Out of scope — please don't file these as bugs

The following are **deliberate non-features** in the current build. Filing these as bugs adds noise. If you have feedback about whether they *should* exist, open a **Discussion** instead.

- **Investments** (holdings, snapshot prices, performance) — planned but not in mobile yet.
- **Reports / charts / insights** — planned but not in mobile yet.
- **Geo-tagging on transactions / a map of where money was spent** — planned but not in mobile yet.
- **Biometric lock, fancy splash, app-icon polish** — planned but not in mobile yet.
- **AI / LLM analysis or narratives in the app** — explicitly out of scope by design (the app must not send financial data anywhere).
- **Sync between two devices, multi-user accounts, household sharing.**
- **Automatic / scheduled / periodic backups, backup rotation, backup encryption.** Backup is a manual user action by design.
- **CSV or JSON import/export.**
- **Recurring transactions, scheduled future transactions, transaction templates.** The app accepts any date the user types but does not project anything forward.
- **Currency conversion.** The user picks one currency; amounts are not converted.
- **Hard-deleting an account or a category.** Only Archive is exposed.

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
