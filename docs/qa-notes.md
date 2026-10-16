# QA notes

Things to know when testing on real devices. Grows with each milestone.

## Reminders (M6)

- **Timing is approximate on purpose.** Reminders use inexact scheduling (the app does not ask
  for `SCHEDULE_EXACT_ALARM`). Android may deliver them several minutes late, more in Doze
  (screen off, unplugged, still) or with battery saver on.
- **Battery optimisation.** Some manufacturers (Xiaomi/MIUI, Huawei/EMUI, Oppo/ColorOS, Vivo,
  Samsung "deep sleeping apps") stop apps they consider unused. Reminders may then stop until the
  app is opened again. Test on at least one such phone; if testers report missing reminders,
  ask them to set the app's battery usage to "Unrestricted" / remove it from sleeping apps.
- **A week ahead.** The next 7 days are scheduled each time the app opens, after every change,
  and by a daily background task (WorkManager, best effort). A user who does not open the app
  for more than a week, on a phone that also blocks background work, stops getting reminders.
- **After a reboot** scheduled reminders are restored by the system (`RECEIVE_BOOT_COMPLETED`).
- **Today's reminder disappears once the day's plan is complete**, and days with an empty plan
  get no reminder.
- **Permission denied:** the app works normally; Settings › Daily reminder explains how to allow
  notifications. The permission is requested only in onboarding (step 4) or when turning the
  reminder on in Settings.
- **Prayer times are approximate** (calculated offline with `adhan`). Compare a few cities with
  local timetables; differences of 1-3 minutes are expected (local authorities add margins).
- Development builds: opening the app from a notification may show a broken splash screen; this
  is a known Expo issue in debug builds only.

## Checklist

- [ ] Onboarding step 4: after Fajr / Dhuhr / Asr / Maghrib / Isha, set time, no reminder
- [ ] City search in Arabic, French and English, accents ignored ("setif" finds Sétif)
- [ ] Custom coordinates accepted / rejected (latitude > 90)
- [ ] Reminder arrives on an Android 13+ device after granting permission
- [ ] No reminder after denying permission, app otherwise unaffected
- [ ] Reminder still scheduled after a reboot
- [ ] Completing today's plan removes today's reminder

## Sync, account and backup (M8)

Automated: engine and account choices against an in-memory server, and `npm run sync:itest`
against a local Supabase (large history, two devices, duplicates, RLS, account deletion). On the
emulator, the app synced its data to a local Supabase (pages, reviews, settings, completed day).

Still to check by hand on a real phone (the development emulator was too overloaded):

- [ ] Settings > Export a backup file: the share sheet opens; save to Files / Drive.
- [ ] Settings > Import a backup file: pick the file; Combine and Replace both work; a random JSON
      file is refused with a clear message.
- [ ] Settings > Delete my account and data: confirmation; the app returns to onboarding; the user
      is gone from Supabase (Authentication > Users). Offline: a clear "nothing was deleted" message.
- [ ] Continue with Google (needs the Google provider set up, runbook step 3): links the anonymous
      account (same user id in Supabase, now with a Google identity) and data stays.
- [ ] Second phone, same Google account: the three choices (Combine / keep account / keep phone).
- [ ] Airplane mode: review several pages, reconnect: they reach the server within seconds.
- [ ] Today's backup card: appears after the first review; "Not now" hides it for good.

Development builds can show "isn't responding" on an overloaded emulator: the traces point at
React Native's debugger connection and debug-only Kotlin reflection at startup, neither present
in release builds. Re-check responsiveness on the release build (M11).
