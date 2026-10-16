# Release checklist

From a finished build to a staged production rollout. Details for each backend step are in
`docs/runbook.md`; Play Console answers are in `docs/data-safety.md`, `docs/app-content.md` and
`docs/store-listing.md`.

## A. Once, before the first upload

### Placeholders and decisions

- [ ] Choose the contact email and replace every placeholder: `grep -rn "CONTACT\|ADRESSE DE CONTACT\|عنوان البريد" docs/`
      (privacy policy ×3, account-deletion page, store listing, tester briefing).
- [ ] Decide the open points in `docs/decisions.md` (sections M7 to M12), in particular backups on the
      Supabase free plan.

### Public pages (GitHub Pages)

The privacy policy and account-deletion page must be public URLs. GitHub Pages on a free account
needs a **public** repository.

- [ ] Either make `slimredtech/hifz` public, or (if the code should stay private) copy `docs/privacy/`,
      `docs/account-deletion.md`, `docs/index.md` and `docs/_config.yml` to a separate public
      repository named `hifz` (same URLs) or update `src/about/links.ts` and these documents.
- [ ] Settings > Pages: source "Deploy from a branch", branch `main`, folder `/docs`.
- [ ] Open https://slimredtech.github.io/hifz/privacy/, `/privacy/fr/`, `/privacy/ar/` and
      `/delete-account/`; check that the Arabic pages read right to left. Internal docs are excluded
      in `docs/_config.yml`.

### Backend (runbook "First-time setup" and "Premium")

- [ ] Supabase project created (linked: `eu-west-1`), migrations pushed (`npx supabase db push`),
      Edge Functions `delete-account` and `revenuecat-webhook` deployed with their secrets.
- [ ] Anonymous sign-ins and manual linking on, Google provider configured, redirect URL added.
- [ ] GitHub secrets for the keep-alive Action set; run it once by hand.
- [ ] `.env.local` holds the production Supabase URL, publishable key and RevenueCat public key.
- [ ] `npm run db:test` and `npm run sync:itest` pass against the local stack.

### Signing

- [ ] Create the upload key (runbook "Release build"), back it up outside the repository (password
      manager plus an offline copy).
- [ ] Add the four `HIFZ_UPLOAD_*` properties to `~/.gradle/gradle.properties`
      (`plugins/withReleaseSigning.js`). Never commit them.
- [ ] Play Console: create the app ("Hifz Planner", default language, free app with in-app
      purchases) and enrol in **Play App Signing**.

### Play Console content

- [ ] App content: every declaration in `docs/app-content.md`, data safety from
      `docs/data-safety.md`, content rating questionnaire.
- [ ] Store listing in en-US, fr-FR and ar from `docs/store-listing.md`.
- [ ] Graphics: icon 512×512, feature graphic 1024×500, at least 4 phone screenshots per language
      (the 8 captions are in the store listing). These are still to be made (app icons are Expo
      defaults until the store assets exist).
- [ ] Monetization: products `premium_monthly`, `premium_yearly` (7-day trial), `premium_lifetime`;
      license testers added (Settings > License testing); RevenueCat connected (runbook "Premium").

## B. Every release

1. [ ] Bump `version` (user-visible, e.g. 1.0.1) and `android.versionCode` (+1, never reused) in
       `app.json`.
2. [ ] `npm run check` passes (types, lint, format, secrets, tests). After package changes:
       `npm run licenses`.
3. [ ] Build: `npx expo prebuild --platform android --clean` then
       `cd android && ./gradlew bundleRelease` → `android/app/build/outputs/bundle/release/app-release.aab`.
4. [ ] `./gradlew :app:signingReport`: variant `release` uses config `release` (the upload key).
5. [ ] `node scripts/check-secrets.js android/app/build/generated/assets` passes.
6. [ ] Upload to **Internal testing** first; install from Play on a phone; quick pass of
       `docs/qa-checklist.md` sections 1, 2 and 8.
7. [ ] Release notes in the three languages (short, factual).

## C. Closed test (new personal developer accounts)

- [ ] Promote the build to **Closed testing**; add at least **12 testers** (Google group or email list).
- [ ] Send `docs/tester-briefing.md` (Arabic / French / English) with the opt-in link and contact.
- [ ] Testers stay opted in **14 consecutive days**; count from the day the 12th tester joins.
- [ ] During the test: full `docs/qa-checklist.md` on the device matrix; fix issues in new builds
      (each build: section B).
- [ ] After 14 days: run `supabase/queries/retention.sql` (D1, D7, reviews per active day, share of
      users with a 7-day streak); collect feedback; apply for production access in Play Console
      (questions about the test, testers' engagement, readiness).

## D. Production

- [ ] Production release with a **staged rollout of 20%**.
- [ ] Watch for 2–3 days: Android vitals (crashes, ANRs), reviews, support email, Supabase usage
      (Reports) and RevenueCat dashboard (purchases reach `entitlements`).
- [ ] No regressions: increase to **100%**. Otherwise halt the rollout, fix, and release a new
      version (section B).
- [ ] After launch: keep the keep-alive Action running while on the free plan; decide on backups
      (runbook) before the user base grows.
