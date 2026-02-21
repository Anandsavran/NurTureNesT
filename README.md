# NurTureNesT  
NurtureNest is a small parenting helper app focused on quick notes, activity logging, baby milestones and short daily tips delivered as system notifications. It combines a simple UI with local persistence and scheduled notifications so parents can get bite-sized reminders and keep small records.

What the app does (feature list)

Home / Quick note — lets the user type a short note and update the screen text. 

Log Activity — opens a LogActivity to create a persisted activity/log (prefill supported).
 
View Logs — opens LogsActivity to view saved logs. 

Charts — ChartsActivity that shows sample line chart data using MPAndroidChart.
 
Daily Tips (notifications) — a toggle to enable/disable automatic daily tips:

When toggled ON the app shows an immediate test notification and schedules daily notifications (TipReceiver) at the configured time (default 9:00).

Each tip is also saved to local history so it can be viewed in-app.

Tips History — new TipsActivity shows previously delivered tips (saved locally).

Milestones — a screen to add/view milestones; currently stored in SharedPreferences as JSON (ListView).

Theme toggle — switch between dark/light via ThemeHelper (persists selection and applies via AppCompatDelegate).

Language toggle — switch app language via LocaleHelper (persisted in SharedPreferences).

Bottom navigation — quick navigation between Home, Logs, Feedback.

How each feature works (technical summary)
Notifications & Daily Tips

NotificationHelper: central helper that creates a notification channel (Android 8+) and shows notifications using NotificationManagerCompat. It safely catches SecurityException so the app won’t crash if notification permission is missing.

TipReceiver: BroadcastReceiver that picks a random tip, shows a system notification via NotificationHelper.showNotification(...), and then writes the tip + timestamp to local storage via TipsRepository.

TipsHelper: schedules/cancels repeating alarms using AlarmManager and a PendingIntent to TipReceiver. By default code uses setInexactRepeating (battery friendly) and schedules at 09:00 or a stored hour/minute. BootReceiver re-schedules the alarm after device reboot if daily tips were enabled.

Permission handling: On Android 13+ (TIRAMISU) the app requests POST_NOTIFICATIONS at runtime; MainActivity contains requestNotificationPermissionIfNeeded() and the test-notification function checks permission before sending.

Flow when user toggles the Daily Tips switch:

Switch ON:

showTestNotification() displays an immediate notification (so the user can verify notifications are allowed).

TipsHelper.enableDailyTip(...) persists enabled + scheduling time and schedules the repeating AlarmManager alarm to TipReceiver.

TipReceiver runs at scheduled time (or immediately on test):

Creates channel (if needed), shows a notification, and calls TipsRepository.addTip(...) to store the tip.

TipsActivity reads stored tips from TipsRepository (SharedPreferences JSON) and displays them in a RecyclerView.

Tips history persistence

TipsRepository: wraps SharedPreferences JSON array:

addTip(context, text, ts) prepends new tip to a JSON array to keep latest first.

getTips(context) returns a MutableList<Tip> with text + timestamp.

clear(context) clears stored tips.

Milestones

Current MilestonesActivity uses a ListView and saves a JSON array of strings to SharedPreferences. (You discussed upgrading this to a Milestone data model with done/timestamp — that’s recommended; see “Next steps”.)

Charts

ChartsActivity uses MPAndroidChart (LineChart) and plots some sample Entry values. Styling uses theme colors.

Theme & Locale

ThemeHelper: stores dark_mode boolean in SharedPreferences, applies AppCompatDelegate night mode.

LocaleHelper: persists app_lang in SharedPreferences, applies locale via createConfigurationContext (API >= N) or updateConfiguration on older APIs.

Where you will see tips (how to test)

Flip the Daily Tips switch ON in the main screen:

You should immediately get a test notification in the system tray (if notification permission was granted).

Later (at the scheduled time — default 09:00), your device will show another notification with a random tip.

In the app, open View Tips (new TipsActivity). That screen shows all tips saved so far (the test one and scheduled ones after they run). You can clear history from that screen.

Storage & persistence summary

Tips: saved to SharedPreferences as a JSON array via TipsRepository.

Milestones: currently saved as a JSON array of strings in SharedPreferences via MilestonesActivity.

Theme & Locale & Tip enable/time: persisted in SharedPreferences (shared key file nurture_prefs).

(You currently have Room in Gradle dependencies, but it's not used yet — good for future DB migration.)

Permissions & System integration

Requires POST_NOTIFICATIONS runtime permission on Android 13+. The app asks for it; if denied, notifications are suppressed gracefully.

Requires RECEIVE_BOOT_COMPLETED in the manifest so BootReceiver can reschedule the alarm after reboot.

Notifications rely on a Notification Channel (created by NotificationHelper) for Android 8+.

UX/interaction notes

You get an immediate visible feedback when enabling tips (test notification).

Tips are both system notifications (so user sees them outside the app) and kept in-app for reference.

Buttons and switch are inside a NestedScrollView so bottom nav doesn’t cover controls.

BottomNavigationView allows quick access to other screens.

Future scope (practical roadmap & suggestions)

Short-term improvements (high impact, low effort)

Fix Milestones model: change milestones to a proper Milestone data class with done: Boolean and timestamp: Long and update adapter + persistence so checkbox toggles are stored with completion date.

Allow user to choose tip time: add UI to pick hour/minute; store in TipsHelper prefs and schedule accordingly.

Switch AlarmManager → WorkManager: use PeriodicWorkRequest for reliable background tasks across Doze and modern Android policies; WorkManager also survives reboots when configured correctly.

Add Settings screen: allow toggling sound/vibration, tip frequency (daily/weekly), channel importance.

Better storage: migrate logs/milestones/tips to Room database (already included in gradle). This gives queries, relational data, and easier migrations.

Notification actions: add an action to notification (e.g., “Save as done”, “Open tip details”) using PendingIntents.

Analytics / Telemetry (optional): track which tips are tapped/saved (careful about privacy).

Unit/UI tests: add instrumentation and unit tests for critical flows (Tip scheduling, permission handling).

Accessibility: ensure color contrast, content descriptions, talkback support, proper touch targets.

Remote tips & personalization: optionally sync tips from a server or allow the user to import their own tips, or surface tips based on baby age.

Long-term / product-level ideas

Profile & baby data: record baby DOB to present age-appropriate tips and milestone recommendations.

Sync & Backup: backup data to cloud or allow export/import (encrypted if sensitive).

Community content: let users share milestone templates or tips (with moderation).

Reminders and tasks: integrate scheduled one-off reminders (vaccinations, appointments).

Monetization: premium tip packs, advanced analytics, exportable milestone timeline (PDF).

Potential pitfalls & what to watch for

AlarmManager.setRepeating is less reliable under Doze/optimizations — use WorkManager for better real-world reliability.

Permissions can be denied — ensure the app handles the denial elegantly (you already catch SecurityException).

If you migrate to Room, carefully plan migrations to avoid data loss.

Keep notification channel settings flexible so users can change importance.

Quick checklist to confirm everything works now

Build & run app on device/emulator.

Grant POST_NOTIFICATIONS when prompted (Android 13+).

On main screen, toggle Daily Tips ON:

You should see an immediate notification.

Open View Tips to see that tip recorded.

Optionally: use TipsHelper.scheduleDailyTip(context, hour, minute) in code or schedule for near future to test scheduled firing.

Reboot device to test BootReceiver re-scheduling (only needed for full production test).
