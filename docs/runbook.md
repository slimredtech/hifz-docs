# Runbook: Supabase backend

How to set up, change and look after the backend. The app is offline-first: if Supabase is down,
paused or not configured, everything works on the phone and data syncs later.

## Free-tier facts (checked 2026-10-15 on supabase.com/pricing; re-check before launch)

| Limit                         | Free plan                                                       |
| ----------------------------- | --------------------------------------------------------------- |
| Monthly active users          | 50,000 (anonymous users count)                                  |
| Database size                 | 500 MB (~600 page rows + a few KB of logs per user: ample)      |
| Egress                        | 5 GB / month                                                    |
| Inactivity                    | **Paused after 1 week** without activity (see keep-alive below) |
| Backups                       | **None** on the free plan (daily backups start with Pro)        |
| Active projects               | 2 per organisation                                              |
| Built-in email                | Heavily rate-limited; not used (Google-only sign-in)            |
| Anonymous sign-ins (per IP/h) | 30 by default; raise in Auth settings if testers share Wi-Fi    |

## First-time setup (when the project exists)

1. Create the project in an EU region (the linked project is in **eu-west-1, Ireland**; the
   privacy policy names Ireland, so update it if the region changes).
2. Authentication > Sign In / Providers: enable **Anonymous sign-ins** and **Manual linking**;
   disable email sign-up. URL Configuration > Redirect URLs: add `hifzplanner://auth-callback`.
3. **Google** (the only sign-in method):
   - Google Cloud Console > APIs & Services > Credentials > Create OAuth client ID >
     **Web application**. Authorized redirect URI: `https://<project-ref>.supabase.co/auth/v1/callback`.
   - OAuth consent screen: app name "Hifz Planner", support email, privacy policy URL; scopes
     `email`, `profile`, `openid` only (no verification needed for these).
   - Supabase > Authentication > Providers > Google: enable, paste the client ID and secret.
   - The app opens Google in a browser tab (Custom Tab), so no Android client ID or SHA-1 is needed.
4. Project Settings > API: copy the **Project URL** and the **publishable key** (`sb_publishable_…`,
   or the legacy `anon` key) into `.env.local` (see `.env.example`). Never copy the secret /
   service_role key anywhere in the app.
5. Apply the schema and deploy the account-deletion function:
   ```bash
   npx supabase login                       # opens a browser once
   npx supabase link --project-ref <ref>    # ref from Project Settings > General
   npx supabase db push                     # applies supabase/migrations/*
   npx supabase functions deploy delete-account
   ```
   (Premium adds a second function, see "Premium" below.)
6. GitHub > Settings > Secrets and variables > Actions: add `SUPABASE_URL` and
   `SUPABASE_PUBLISHABLE_KEY`, then run "Supabase keep-alive" once by hand to check it.
7. Rebuild the app (`npm run android`); `EXPO_PUBLIC_*` values are compiled into the bundle.

## Premium (RevenueCat, M9)

1. Play Console > Monetize > Products: subscriptions `premium_monthly`, `premium_yearly` (base plan
   with a 7-day free trial offer) and the in-app product `premium_lifetime`.
2. RevenueCat: create the project and the Android app (package `com.beyapps.hifzplanner`), connect
   the Play service account, import the three products, entitlement **`premium`** attached to all
   three, offering **`default`** with Monthly, Annual and Lifetime packages.
3. Put the RevenueCat **public** Android SDK key (`goog_…`) in `.env.local` as
   `EXPO_PUBLIC_REVENUECAT_ANDROID_KEY` and rebuild. Without it the Premium screen says purchases
   are unavailable and everything else works.
4. Webhook: generate a long random secret, then
   ```bash
   npx supabase secrets set REVENUECAT_WEBHOOK_SECRET=<random> REVENUECAT_SECRET_API_KEY=<sk_…>
   npx supabase functions deploy revenuecat-webhook --no-verify-jwt
   ```
   RevenueCat > Integrations > Webhooks: URL
   `https://<project-ref>.supabase.co/functions/v1/revenuecat-webhook`, Authorization header
   `Bearer <random>`. Send a test event: it returns 200 and changes nothing (test user ids are ignored).
5. Test with a Play license tester: buy, check `entitlements` in the table editor, cancel, check
   it expires. The RevenueCat **secret** key lives only in Supabase secrets, never in the app.

## Changing the schema

1. Add a new file in `supabase/migrations/` (never edit an applied one):
   `npx supabase migration new <name>`.
2. Every new table: enable RLS, add policies, and **grant** privileges explicitly. Default
   privileges are revoked, so a forgotten grant fails closed ("permission denied").
3. Add pgTAP tests in `supabase/tests/database/` and run `npm run db:start` then `npm run db:test`.
4. `npx supabase db push` to apply to the hosted project.

## Local development

- `npm run db:start`: local Supabase in Docker (Docker Desktop must be running).
- `npm run db:test`: RLS / security tests.
- `npm run sync:itest`: the sync engine and the delete-account function against the local stack
  (`npx supabase migration up --local` first if the local database predates a migration).
- `npm run db:reset`: rebuild the local DB from the migrations. `npx supabase stop` when done.

## If the project is paused

The keep-alive Action calls `public.ping()` every 3 days. If the project paused anyway (e.g.
secrets missing), open the dashboard and click **Restore**; nothing is lost and phones sync
again on their own. Check the Action's run history afterwards.

## Backups (free plan has none)

User data lives on each phone first, so a server loss is recoverable as users re-sync, but
retention data and teacher links would be lost. Options, to decide before launch:

- Weekly `npx supabase db dump --data-only` by hand, stored encrypted off-GitHub.
- A scheduled Action that dumps the database (needs the database password as a secret; the dump
  contains user data, so it must be encrypted and the privacy policy must mention it).
- Upgrade to Pro (daily backups) once the app has real users.

**Restore** a dump into a fresh project: create project, `db push` the migrations, then
`psql "$DATABASE_URL" -f dump.sql`.

## Rotating keys

- **Publishable key** (public): create a new one in Project Settings > API, update `.env.local`
  and the GitHub secret, ship an app update, then revoke the old key once few users still run the
  old version.
- **Secret / service_role key** (Edge Functions only, from M8): roll it in the dashboard and
  update the function secrets (`npx supabase secrets set ...`). It never ships in the app;
  `npm run check:secrets` fails if one appears in the repo or a bundle.

## Retention numbers

Run `supabase/queries/retention.sql` in the SQL editor: D1/D7/D14/D30 by cohort, average reviews
per active day, and the share of users who reached a 7-day streak.

## Account deletion

- In the app: Settings > Account and backup > Delete my account and data (Edge Function
  `delete-account`, which deletes the auth user; every table cascades).
- Public page for the Play Console's "delete account URL": `docs/account-deletion.md`, published
  with GitHub Pages (Settings > Pages > branch `main`, folder `/docs`). **Replace the contact email
  placeholders before publishing.**
- Email requests: delete the user in Authentication > Users (cascades all data) within 30 days and
  reply to confirm.

## Release build (M11)

Release builds are local (no EAS queue): Hermes, R8 minify and resource shrinking are on
(`expo-build-properties` in `app.json`).

1. Put the production values in `.env.local` (Supabase URL and publishable key, RevenueCat public
   key); they are compiled into the bundle.
2. Signing: create the **upload key** once and keep it outside the repository, with a backup:
   ```bash
   keytool -genkeypair -v -storetype PKCS12 -keystore hifz-upload.keystore \
     -alias hifz-upload -keyalg RSA -keysize 2048 -validity 10000
   ```
   Enrol in **Play App Signing** when creating the app in Play Console (Google keeps the app
   signing key; a lost upload key can be reset). Then add to `~/.gradle/gradle.properties`
   (never to the repository):
   ```
   HIFZ_UPLOAD_STORE_FILE=/absolute/path/hifz-upload.keystore
   HIFZ_UPLOAD_STORE_PASSWORD=...
   HIFZ_UPLOAD_KEY_ALIAS=hifz-upload
   HIFZ_UPLOAD_KEY_PASSWORD=...
   ```
   `plugins/withReleaseSigning.js` signs release builds with it; without these properties, release
   builds fall back to the debug key (testing only, Play refuses them). Check with
   `./gradlew :app:signingReport` (variant `release` must show config `release`).
3. Build:
   ```bash
   npx expo prebuild --platform android
   cd android && ./gradlew bundleRelease     # AAB for Play; assembleRelease for a test APK
   ```
4. Check before uploading: `npm run check`, then scan the built bundle with
   `node scripts/check-secrets.js android/app/build/generated/assets`, install the APK on a device and confirm there is
   no dev menu (shake / `adb shell input keyevent 82` does nothing).
5. After adding or updating a package: `npm run licenses` (the About screen's list; a test
   fails if it is stale).
