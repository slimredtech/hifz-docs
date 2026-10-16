# Play Console: Data safety answers

Copy these into Play Console > App content > Data safety. They match the code as of M12 and
`docs/privacy/` (privacy policy) and `docs/dependencies.md` (SDK audit). Re-check them whenever a
package or a synced field is added.

## Data collection and security

| Question                                                              | Answer                                                                                                                                                  |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Does your app collect or share any of the required user data types?   | **Yes**                                                                                                                                                 |
| Is all of the user data collected by your app encrypted in transit?   | **Yes** (HTTPS only: Supabase, RevenueCat, Google)                                                                                                      |
| Do you provide a way for users to request that their data is deleted? | **Yes**: in the app (Settings > Delete my account and data) and at https://slimredtech.github.io/hifz/delete-account/                                   |
| Does your app allow users to create an account?                       | **Yes**. Methods: **OAuth** (Sign in with Google, optional). An anonymous account is also created automatically at first launch, without any user input |
| Delete account URL                                                    | https://slimredtech.github.io/hifz/delete-account/                                                                                                      |

## Data types

"Collected" = sent off the device. Nothing below is **shared**: Supabase, RevenueCat and Google
act as service providers processing data on our behalf, and data seen by a teacher or partner is
transferred at the user's own request after a consent screen, both of which Play excludes from
"sharing". No data is processed only ephemerally.

| Category                                                                                   | Data type                                                                            | Collected | Required or optional | Purposes                              | Notes                                                                                                                           |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ | --------- | -------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Personal info                                                                              | **User IDs**                                                                         | Yes       | Required             | App functionality, Account management | Random account ID created at first launch; also the RevenueCat app user ID                                                      |
| Personal info                                                                              | **Email address**                                                                    | Yes       | Optional             | Account management                    | Only if the user links Google                                                                                                   |
| Personal info                                                                              | **Name**                                                                             | Yes       | Optional             | App functionality, Account management | Optional display name for teacher/partner; the name in the Google profile if Google is linked                                   |
| Financial info                                                                             | **Purchase history**                                                                 | Yes       | Optional             | App functionality                     | Premium status and purchase records (Google Play, RevenueCat)                                                                   |
| App activity                                                                               | **App interactions**                                                                 | Yes       | Required             | App functionality, Analytics          | Page progress, review history (date, rating), days used; retention measured in aggregate only                                   |
| App activity                                                                               | **Other user-generated content**                                                     | Yes       | Optional             | App functionality                     | A teacher's weekly note (max 140 characters) to a linked student                                                                |
| Personal info                                                                              | Address, phone, race, religious or political beliefs, sexual orientation, other info | **No**    | –                    | –                                     | The app does not ask about or record religious belief; it records memorization progress only                                    |
| Location                                                                                   | Approximate / precise                                                                | **No**    | –                    | –                                     | The reminder's city or coordinates stay on the phone (`src/sync/deviceOnlySettings.ts`); prayer times are computed on the phone |
| Financial info                                                                             | Payment info, credit score, other                                                    | **No**    | –                    | –                                     | Payments are handled by Google Play                                                                                             |
| Health and fitness, Messages, Photos and videos, Audio, Files and docs, Calendar, Contacts | –                                                                                    | **No**    | –                    | –                                     |                                                                                                                                 |
| App info and performance                                                                   | Crash logs, diagnostics                                                              | **No**    | –                    | –                                     | No crash SDK; Android vitals is Google's own                                                                                    |
| Device or other IDs                                                                        | –                                                                                    | **No**    | –                    | –                                     | No advertising ID, Android ID or device ID is read                                                                              |
| Web browsing                                                                               | –                                                                                    | **No**    | –                    | –                                     |                                                                                                                                 |

### Purposes, per Play's definitions

- **App functionality:** backup, sync between phones, scheduling, streak, teacher and partner
  features, unlocking Premium.
- **Account management:** anonymous account, linking Google, account deletion.
- **Analytics:** aggregate retention figures (D1/D7/D14/D30, reviews per active day) computed in
  SQL on `daily_activity` (`supabase/queries/retention.sql`); no third-party analytics.
- Not used for: advertising or marketing, fraud prevention, personalisation, developer
  communications.

## Points to review before submitting

- **"Name" and the Google profile:** Supabase Auth stores the name and picture URL Google returns
  with the sign-in. The app never displays or uses them, but they are stored, so "Name" is
  declared. Re-check if the Google scopes change (currently `openid email profile`).
- **IP addresses:** Supabase processes request IPs and keeps short-lived logs for security. Play
  treats IP addresses as collected only when used to derive location or identify users; we do
  neither, so they are not declared. Mention this in the review notes if asked.
- **Religious beliefs:** Play's "Religious or philosophical beliefs" type is for data that states
  a belief. The app stores which Quran pages are memorized, not the user's beliefs, so it is not
  declared; this is a judgement call to confirm.
