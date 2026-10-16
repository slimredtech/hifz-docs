# Manual QA checklist (release candidate)

Run on a **release build** signed with the upload key (`docs/release-checklist.md`), installed
from the Play internal or closed testing track when possible. Background notes per feature:
`docs/qa-notes.md`. Tick every box on at least one device; the device matrix says which.

## Device matrix

| Device                                 | Android | Why                                                     | Done |
| -------------------------------------- | ------- | ------------------------------------------------------- | ---- |
| Low-end phone (2–3 GB RAM, ~5" screen) | 7 or 8  | Oldest supported (minSdk 24 = Android 7.0), performance | [ ]  |
| Mid-range Samsung or Xiaomi            | 11      | Aggressive battery management (reminders)               | [ ]  |
| Recent Pixel or equivalent             | 13      | Notification permission                                 | [ ]  |
| Any phone                              | 14+     | Latest behaviour                                        | [ ]  |
| Tablet or large screen (optional)      | any     | Layout                                                  | [ ]  |

## 1. First run and onboarding

- [ ] Fresh install: language follows the phone (Arabic phone → Arabic, right-to-left).
- [ ] The 4 steps: language, pages already memorized (surah picker), daily target and direction,
      reminder. Back works on each step.
- [ ] "Just starting" leads to page 1 (or 604 when memorizing backward) as the first new page.
- [ ] The notification permission is asked only at the reminder step.
- [ ] Today appears with a plan; no crash when killing the app during onboarding and reopening.

## 2. Daily use

- [ ] Mark the new page memorized, rate a revision Good / Hard / Forgot; Undo works each time.
- [ ] The progress ring and streak update; the day completes; the streak counts it the next day.
- [ ] Skip a day: a grace day is used (2 a month free). Skip three: the streak resets.
- [ ] Map: overview and pages views, pinch between them, page sheet actions.
- [ ] Stats: charts, calendar, weakest pages; empty state on a fresh install.
- [ ] Share progress: the share sheet opens with the image; nothing personal on it.
- [ ] Widget: add it from the home screen; it updates after a review and the next day.

## 3. Languages, display and accessibility

- [ ] Arabic, French, English: every screen, no clipped or untranslated text; switching between
      Arabic and a left-to-right language restarts the app in the right direction.
- [ ] Dark mode and light mode (phone setting and in-app choice).
- [ ] Font size at the largest setting (≥130%) and display size large: nothing cut off.
- [ ] TalkBack: every button and page cell is read with its meaning; charts are read as a summary.
- [ ] Pattern mode on: every page state has a distinct mark on the map, legend and Today.

## 4. Reminders

- [ ] After a chosen prayer, in at least two cities; at a fixed time.
- [ ] Still delivered after a reboot and after 2 days without opening the app.
- [ ] Not delivered once the day's plan is complete.
- [ ] Permission denied: the app works; Settings explains how to allow notifications.
- [ ] Battery saver on: note any delay (expected; see qa-notes).

## 5. Offline, sync and account

- [ ] Airplane mode for a full day of use: everything works; on reconnecting, data syncs.
- [ ] Slow network (throttled Wi-Fi): no freeze; sync retries quietly.
- [ ] Continue with Google: same account keeps its data; the email appears in Settings.
- [ ] Uninstall, reinstall, Continue with Google: progress comes back.
- [ ] Second phone with the same Google account: the Combine / keep account / keep phone choice.
- [ ] Export a backup file; import it (Replace and Combine); a random file is refused clearly.
- [ ] Delete my account and data: back to onboarding; user gone in Supabase; offline gives a clear
      "nothing was deleted" message.
- [ ] The reminder's city is **not** in the `user_settings` row in Supabase (location stays on the
      phone).

## 6. Premium (license testers, RevenueCat sandbox)

- [ ] Premium screen shows store prices, the trial, renewal terms, Restore and Manage.
- [ ] Buy monthly with a test card: Premium turns on; `entitlements` row updated by the webhook.
- [ ] Cancel: Premium stays until the end of the period, then turns off (test subscriptions renew
      every few minutes).
- [ ] Lifetime purchase; Restore on a reinstall; pending payment ("slow test card") message.
- [ ] Offline for 2 days with Premium: still unlocked (3-day grace).
- [ ] Premium unlocks: 4 grace days, 3 reminders, full stats, HD share image, unlimited students.

## 7. Teacher and partner (two phones)

- [ ] Teacher invites; student joins after the consent screen; wrong / expired / reused codes give
      clear messages; 11 wrong codes in an hour are blocked.
- [ ] Free teacher: second student refused with the limit message.
- [ ] Teacher sets a weekly assignment; it appears on the student's Today; memorizing the range
      marks it done.
- [ ] Student stops sharing: the teacher loses access at once.
- [ ] Partners: each sees the other's streak and "done today" only; remove works from either side.

## 8. Release build checks

- [ ] No dev menu (shake, or `adb shell input keyevent 82`), no debug screen, no red box.
- [ ] `node scripts/check-secrets.js android/app/build/generated/assets` passes.
- [ ] About shows the right version, Tanzil attribution and licences; Privacy policy link opens.
- [ ] Cold start under ~3 s on the low-end phone; scrolling the map and Stats stays smooth.
