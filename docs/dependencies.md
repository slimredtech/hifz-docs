# Dependencies

One line per direct dependency explaining why it is here (PLAN.md section 17), plus whether it
collects or sends any user data (feeds the SDK audit in PLAN-EXPO-SUPABASE.md section 7).
Update this file in the same commit that adds or removes a dependency.

## Runtime

| Package                                                    | Why                                                                                                         | Data collected                                                                                  |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `expo`                                                     | Framework and native module runtime (plan stack, [FIXED])                                                   | None                                                                                            |
| `react`, `react-native`                                    | UI runtime required by Expo                                                                                 | None                                                                                            |
| `expo-router`                                              | File-based navigation with the 4 bottom tabs (plan stack)                                                   | None                                                                                            |
| `react-native-screens`                                     | Native screen containers, required by expo-router                                                           | None                                                                                            |
| `react-native-safe-area-context`                           | Safe-area insets, required by expo-router                                                                   | None                                                                                            |
| `expo-linking`, `expo-constants`                           | Required by expo-router                                                                                     | None                                                                                            |
| `expo-status-bar`                                          | Status-bar style that follows light/dark theme                                                              | None                                                                                            |
| `expo-system-ui`                                           | Required on Android for `userInterfaceStyle: automatic` (dark mode)                                         | None                                                                                            |
| `expo-splash-screen`                                       | Keeps the splash up until language/direction are applied                                                    | None                                                                                            |
| `expo-dev-client`                                          | Development builds (plan: dev builds, not Expo Go)                                                          | None (dev only)                                                                                 |
| `expo-localization`                                        | Device language detection and RTL support config                                                            | None                                                                                            |
| `i18next`, `react-i18next`                                 | Translations ar/fr/en (plan stack)                                                                          | None                                                                                            |
| `expo-font`                                                | Embeds the Arabic font at build time                                                                        | None                                                                                            |
| `expo-asset`                                               | Required peer of expo-font                                                                                  | None                                                                                            |
| `@expo/vector-icons`                                       | Tab and UI icons (bundled icon fonts, no network)                                                           | None                                                                                            |
| `expo-sqlite`                                              | Local database, source of truth (plan stack); its key-value store holds display preferences                 | None                                                                                            |
| `zustand`                                                  | UI state (plan stack)                                                                                       | None                                                                                            |
| `drizzle-orm`                                              | Typed queries and migrations over expo-sqlite (plan stack). Stable 0.45, not the 1.0 RC                     | None                                                                                            |
| `expo-crypto`                                              | `randomUUID()` for review-log ids, so sync uploads are idempotent                                           | None                                                                                            |
| `babel-preset-expo`                                        | Required by Expo once `babel.config.js` exists (added via `npx expo customize`)                             | None (build)                                                                                    |
| `@shopify/react-native-skia`                               | Draws the 604-cell juz overview as one picture (plan stack). Adds several MB to the APK                     | None                                                                                            |
| `react-native-gesture-handler`                             | Pinch to switch between juz view and page view                                                              | None                                                                                            |
| `react-native-reanimated`, `react-native-worklets`         | Required at runtime by Skia 2.6 on native, despite being listed as an optional peer (found on the emulator) | None                                                                                            |
| `adhan`                                                    | Offline prayer-time calculation (plan stack, MIT)                                                           | None                                                                                            |
| `expo-notifications`                                       | Local daily reminder only; no push, no tokens requested                                                     | None                                                                                            |
| `expo-background-task`, `expo-task-manager`                | Daily best-effort refresh of the reminder week (WorkManager)                                                | None                                                                                            |
| `@supabase/supabase-js`                                    | Supabase client: anonymous auth now, sync (M8). Talks only to our Supabase project                          | Anonymous user id; activity days and review counts (M7); progress (M8), see docs/data-safety.md |
| `expo-web-browser`                                         | Google sign-in in a browser tab (Custom Tab), PKCE                                                          | None                                                                                            |
| `expo-file-system`, `expo-sharing`, `expo-document-picker` | Manual backup export (share sheet) and import (file picker); no storage permission                          | None                                                                                            |
| `expo-network`                                             | Sync again when the connection comes back                                                                   | None                                                                                            |
| `react-native-purchases` (RevenueCat)                      | Premium purchases through Google Play Billing, restore, entitlement status                                  | Supabase user id (as RevenueCat app user id) and purchase history; to list in Data Safety       |
| `react-native-view-shot`                                   | Captures the progress card as a PNG in the cache directory for the share sheet (no storage permission)      | None                                                                                            |
| `react-native-android-widget`                              | Home-screen Today widget (Expo config plugin), rendered from the local database                             | None                                                                                            |
| `expo-build-properties`                                    | Build-time only: turns on R8 and resource shrinking for release builds                                      | None                                                                                            |

## Development only

| Package                                                                                        | Why                                                                   |
| ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `typescript`, `@types/react`, `@types/jest`                                                    | Strict type checking                                                  |
| `jest`, `jest-expo`, `@react-native/jest-preset`                                               | Unit tests (plan stack)                                               |
| `@testing-library/react-native`, `test-renderer`                                               | Screen tests (plan stack)                                             |
| `react-dom` (pinned to the React version)                                                      | Peer of Expo's test/server tooling; not bundled in the app            |
| `eslint`, `eslint-config-expo`, `prettier`, `eslint-config-prettier`, `eslint-plugin-prettier` | Lint and formatting                                                   |
| `supabase` (CLI)                                                                               | Local Supabase, migrations and pgTAP tests (`npm run db:*`)           |
| `drizzle-kit`                                                                                  | Generates SQL migrations from the schema (`npm run db:generate`)      |
| `babel-plugin-inline-import`                                                                   | Bundles Drizzle's `.sql` migrations as strings (Drizzle's Expo guide) |
| `better-sqlite3` (v12, Node 20 support), `@types/better-sqlite3`                               | In-memory SQLite for repository tests, running the real migrations    |

## Android permissions (current)

| Permission                                           | Source                                   | Status                                                                |
| ---------------------------------------------------- | ---------------------------------------- | --------------------------------------------------------------------- |
| `INTERNET`                                           | React Native default                     | Kept: needed for Supabase sync (M7+) and RevenueCat (M9)              |
| `VIBRATE`                                            | Expo default                             | Removed in `app.json`; reminders do not vibrate                       |
| `POST_NOTIFICATIONS`                                 | expo-notifications                       | Kept: the one runtime permission (asked in onboarding only)           |
| `RECEIVE_BOOT_COMPLETED`                             | expo-notifications, expo-background-task | Kept: restores scheduled reminders after a reboot (no runtime prompt) |
| `WAKE_LOCK`, `FOREGROUND_SERVICE` (if present)       | WorkManager (background task)            | Library-added; listed for the Data Safety review                      |
| `com.android.vending.BILLING`                        | react-native-purchases (Play Billing)    | Library-added; required for in-app purchases, no runtime prompt       |
| `SCHEDULE_EXACT_ALARM`, `USE_EXACT_ALARM`            | -                                        | Blocked in `app.json`: reminders are inexact by design                |
| `SYSTEM_ALERT_WINDOW`, `READ/WRITE_EXTERNAL_STORAGE` | Expo/React Native defaults               | Removed in `app.json`                                                 |
