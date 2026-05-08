# Finance Tracker — Privacy Policy

**Effective date:** 2026-05-07
**Contact:** ericocamposlira@gmail.com — public bug tracker at https://github.com/ericocampos/finance-tracker-bugs

## Summary

Finance Tracker is a personal, single-user, **local-first** finance app. All of your financial data — accounts, balances, transactions, categories, observations, and any locations you tag — stays on your device. The app has no user accounts, no login, no cloud sync, and no analytics.

We do not collect, transmit, store, sell, or share your personal financial data. Ever.

## What stays on your device

The following data is created and stored exclusively in the app's local SQLite database on your phone, and never leaves it through the app:

- Bank/account names, opening balances, and current balances
- Income, expenses, and transfer records (amount, currency, date, category, observations)
- Categories and category icons
- Locations you optionally attach to a transaction (latitude/longitude)
- App settings (selected language, theme, currency display preferences)

If you uninstall the app, this data is deleted with it. If you want a backup, you must create one yourself using the in-app export feature, which writes a file you control via your operating system's share sheet.

## What leaves your device

The app has no servers of its own and makes no API calls to send your financial data anywhere. The only outbound network activity from the app is:

| What | When | What is sent | What is NOT sent |
| --- | --- | --- | --- |
| **Expo / EAS over-the-air update check** | App launch, periodically | Your app's runtime version, channel name, and a project ID, so the update server can decide whether a new JavaScript bundle is available | None of your financial data, account contents, transactions, locations, or settings |
| **Google Maps tile loads** | Only when you open the in-app **Map** view | Your device's IP address and the tile coordinates Google needs to render the map area you are viewing, governed by Google's own privacy policy | Your transactions, balances, or any other app data |

There is **no** in-app analytics, telemetry, crash reporting, advertising SDK, or third-party tracking. There is no in-app artificial intelligence feature, no API key, and no LLM call that includes your data.

## Permissions the app requests

| Permission | Why | Data handling |
| --- | --- | --- |
| **Location** (`ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`) on Android; `NSLocationWhenInUseUsageDescription` on iOS | Optional. If you choose to "tag with location" while creating a transaction, the app reads your current coordinates and stores them in the local database alongside that transaction so it can later show the transaction on a map | Coordinates are written to the local SQLite database only. They are never uploaded |
| **Storage / file access** (via OS share sheet only) | Used when you export a backup, so the operating system can hand the exported file to an app of your choice (email, cloud drive, messaging) | The destination of the exported file is entirely your choice and is outside this app |

You can deny or revoke any permission in your device settings; the only feature that depends on Location is the optional "tag with location" capture. Everything else continues to work normally without it.

## What we (the developer) collect from you

Outside the app:

- **Crash and performance reports automatically collected by the Google Play Store** (when the app is installed via Play). These are anonymized device-level reports controlled by Play's own policies and your device's "Send usage and diagnostic data" setting. They do not contain your transaction data. You can opt out at any time via Android's settings.
- **Bug reports and emails you choose to send** to the public bug tracker (`github.com/ericocampos/finance-tracker-bugs`) or the contact email above. Whatever you write in those reports is what we receive — please do not paste sensitive data unless it is necessary for reproduction.

That is the entire list. There is no other channel through which the developer receives data about your app usage.

## Children

The app is not directed to children under 13, and we do not knowingly collect data from children. The app can be used by anyone, but it has no features specifically aimed at minors.

## Future paid features

This app is currently free. If a paid tier is introduced later (for example, a one-time license unlocking certain features), payment processing will be handled by Google Play Billing, and the developer would receive only the metadata Google provides about the purchase — typically an obfuscated purchase token, the SKU bought, the country of purchase, and a timestamp. **No financial data from your ledger would ever be transmitted as part of any such purchase.** This policy will be updated before any such feature ships.

## Changes to this policy

If this policy materially changes, the new version will be published at the same URL as the current one, with an updated "Effective date" at the top. The change history is also available in the app's source repository.

## Your rights

Because we receive none of your financial data, there is nothing for the developer to delete, export, or correct on your behalf — your data is already entirely on your device, under your control. To delete it, uninstall the app or use the in-app reset feature in Settings. To back it up, use the in-app export feature.

If you sent us a bug report or email and want it deleted, contact us at the address above.

## Contact

Erico Campos — ericocamposlira@gmail.com
Public bug tracker: https://github.com/ericocampos/finance-tracker-bugs
