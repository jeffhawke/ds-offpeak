# DS Off-Peak — Specification

**Version:** 0.1 — 2026-08-27

## 1. Purpose

A small personal Android app that shows, at a glance, whether DeepSeek API usage is currently billed at the off-peak (cheaper) rate. DeepSeek applies the higher peak rate during two daily windows from Monday to Friday, expressed in UTC. The user is not on the UTC timezone, so this app removes the mental conversion and answers one question: *"can I use DeepSeek at the off-peak rate right now?"*

The app is a tariff-schedule indicator only. It works fully offline: no internet, no accounts, no third-party services, no DeepSeek integration.

## 2. Default tariff schedule (source data)

Taken from the DeepSeek pricing page (https://api-docs.deepseek.com/quick_start/pricing/), confirmed by the user on 2026-08-27:

- Peak (more expensive) windows — **Monday to Friday, UTC**:
  - **01:00 – 04:00 UTC**
  - **06:00 – 10:00 UTC**
- All other hours are off-peak, **including all of Saturday and Sunday**.
- Off-peak rates are exactly half of the peak rates (context only; the app does not display prices).

The schedule is preloaded as the default configuration and is user-editable (§5 F4), because DeepSeek may change these windows over time. "Reset to defaults" restores this schedule.

## 3. Terminology

- **Peak period** — a configured interval during which the peak (expensive) rate applies.
- **Off-peak period** — any time not inside a peak interval.
- **Status** — one of four states:
  | Status | Color | Meaning |
  |---|---|---|
  | OFF-PEAK | green | No peak interval now, and none starting within 30 minutes |
  | PEAK-SOON-30 | yellow | Off-peak now, but the next peak starts within 30 minutes |
  | PEAK-SOON-15 | orange | Off-peak now, but the next peak starts within 15 minutes |
  | PEAK | red | Inside a peak interval |
- All schedule times are expressed in **UTC**. "Now in UTC" is derived from the device clock (epoch time), so timezone and daylight-saving changes never affect the computation.

## 4. Status model

Evaluated continuously (refresh every second):

1. If *now* is inside any configured peak interval → **PEAK** (red).
2. Otherwise, let *t* = start of the earliest peak interval strictly after *now*:
   - *t − now* ≤ 15 min → **PEAK-SOON-15** (orange).
   - *t − now* ≤ 30 min → **PEAK-SOON-30** (yellow).
   - otherwise → **OFF-PEAK** (green).
3. If **no peak intervals are configured at all**, the status is always OFF-PEAK (green) and the countdown is hidden.

The warning thresholds (30 min, 15 min) are **fixed for v0.1** (future idea: make them configurable).

**Countdown shown:**
- PEAK → time until the current peak interval ends ("Peak ends in 1h 23m").
- OFF-PEAK / PEAK-SOON-* → time until the next peak interval starts ("Off-peak ends in 12m").

Formatted as "Xd Yh", "Xh Ym", "Xm", or "Xs" as appropriate, refreshed every second; the status flips exactly at each boundary (peak start, peak end, 30-min mark, 15-min mark).

## 5. Functional requirements

**F1 — Status indicator (core).** The main screen shows the status unmistakably: the whole background (or a large filled area) is colored green / yellow / orange / red, and a text label is always shown alongside ("Off-peak", "Peak in <30 min", "Peak in <15 min", "Peak") so the state is readable even without color.

**F2 — Clocks.** The main screen shows:
- the current UTC time, 24-hour HH:MM:SS, labeled "UTC";
- the device's local time next to it, labeled with the device timezone name/offset (user decision, 2026-08-27).

**F3 — Countdown & next change.** The countdown line (§4), plus the exact UTC wall-clock time of the next status change ("Next change 06:00 UTC").

**F4 — Editable schedule.** Accessible from the settings (options menu):
- For each of the 7 days of the week: a list of peak intervals (start, end), each in UTC; add / remove intervals, set times with a minute-precision time picker.
- Validation rules:
  - end must be strictly after start;
  - intervals must not overlap — overlapping or adjacent intervals are merged automatically when saving;
  - a window crossing midnight is entered as two intervals (e.g. 22:00–24:00 and 00:00–02:00).
- "Reset to defaults" restores the schedule from §2.
- Changes are persisted (survive app restart) and apply to the main screen immediately. If an edit changes the current status, the UI updates right away; the notification rule (F5) still applies only when the app is not in the foreground.

**F5 — Notifications.** A notification fires when the status changes between any of the four states (OFF-PEAK → PEAK-SOON-30 → PEAK-SOON-15 → PEAK → OFF-PEAK), **only when the app is not in the foreground**.
- Content: new status plus the relevant countdown, e.g. "Peak starts in 15 minutes", "Off-peak started".
- Requires the Android 13+ runtime notification permission. If denied, no notifications are shown and the app never nags about it (the settings screen shows the state).
- Notification style is a user setting: **text only**, or **sound + text** (device default notification sound).
- Background behavior is **best effort** (user decision, 2026-08-27): the app schedules transitions as close to on-time as Android allows; when the app is fully closed, some devices may delay a notification by a few minutes. No foreground service.

**F6 — Offline.** The app must behave identically with no network: it declares no INTERNET permission, connects to no third-party service, and performs no network time sync. It relies on the device clock being correct.

## 6. Screens

**6.1 Main screen** (single activity):
- Large colored status area: color + status label (F1).
- UTC clock (HH:MM:SS) and local time with timezone label (F2).
- Countdown line and "Next change at <HH:MM> UTC" (F3).
- Menu button → settings.

**6.2 Settings screen:**
- Peak-hours editor per day of week (F4).
- "Reset to defaults" (F4).
- Notifications: enable / disable; style (text-only / sound+text); shortcut to the system permission page (F5).
- About: note that the schedule is in UTC, the default DeepSeek windows (§2), app version.

## 7. Constraints

- Offline-only: no INTERNET permission, no third-party services, no accounts.
- Single user, personal use; clean, functional, small; English UI text.
- Schedule defined in UTC → immune to DST and timezone changes.
- Relies on the device clock being correct (offline by design, no NTP).
- Lightweight: no analytics, no ads, no background work beyond scheduled notifications.

## 8. Non-goals (v0.1)

- No DeepSeek API integration, login, or usage tracking.
- No home-screen widget (future idea, §10).
- No automatic handling of holidays / special days — the user edits that day's windows manually if needed.
- No price display (prices change; the app tracks windows only).
- No localization / multi-language support.
- No network time sync.

## 9. Acceptance criteria

- **A1** — For any UTC instant and any valid schedule, the main screen shows the correct status and countdown per §4 — including weekends (full off-peak), days with several windows, days with no windows, and windows adjacent or overlapping (merged).
- **A2** — Clock and countdown update every second; the status flips exactly at each boundary (peak start, peak end, 30-min and 15-min marks).
- **A3** — Edited schedules persist across app restarts, are validated per F4, and are reflected immediately.
- **A4** — "Reset to defaults" restores the §2 schedule.
- **A5** — Notifications fire on every status transition when the app is not in the foreground, best effort (per F5); none when the permission is denied or notifications are disabled; correct style (text-only vs sound+text) per settings.
- **A6** — The app installs and runs with zero network access (airplane mode); behavior is identical.
- **A7** — The schedule editor handles: a day with no intervals, several intervals per day, overlapping/adjacent intervals (merged), an invalid interval (end ≤ start) rejected, and a midnight-crossing window entered as two intervals.

## 10. Future ideas (explicitly out of scope for v0.1)

- Home-screen widget with the same indicator.
- Configurable warning thresholds (30/15 min).
- Vibration option alongside sound.
- Display of current off-peak / peak prices (editable static values, still offline).
- Persistent status notification in the notification shade.

## 11. Open items

- Min / target Android SDK versions — to be decided at the implementation-plan stage (AGENTS.md note: build/test details will be added when the app is scaffolded).
