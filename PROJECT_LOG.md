# PROJECT.log

Append-only log of changes to `AGENTS.md`, `spec.md`, and significant project decisions.

---

## 2026-08-25 22:22 — AGENTS.md created (v0.1)

Initial working rules agreed with the user:

- Repository & commit conventions: commit after every completed step, direct on `main`, explicit-path staging only (never `git add .` / `git add -A`).
- `my_considerations.txt` is the user's personal scratch file: never read, edited, staged, or committed by the agent; user changes are ignored silently; stays tracked in git.
- `.gitignore` policy: ignore rebuilt/redownloaded artifacts and local tool state (`.reasonix/`); commit everything essential to rebuild/redevelop, including the Gradle wrapper.
- `spec.md` (project contract): versioned from 0.1, never modified without explicit permission, all changes logged here.
- `AGENTS.md` (these rules): same protection as `spec.md`, versioned, changes logged here.
- `PROJECT.log`: append-only; also records significant project decisions (architecture, dependencies, why-choices) — agreed at creation time.
- Decision rule: consequential choices are submitted to the user before being applied; reversible low-risk choices default to the sensible option.

---

## 2026-08-27 — spec.md created (v0.1)

First specification draft for the app (working title "DS Off-Peak"): a personal Android indicator that shows, at a glance, whether DeepSeek API usage is currently at off-peak (cheaper) rates.

Acceptance criteria (spec.md §9): A1 status/countdown correctness incl. weekends & window merging; A2 per-second updates & exact boundary flips; A3 persisted+validated+immediate schedule edits; A4 reset-to-defaults; A5 best-effort notifications on every transition only when backgrounded, respecting permission + style; A6 offline/airplane behavior identical; A7 schedule editor edge cases.

Decisions recorded (user-approved via interactive questions, 2026-08-27):

- **Default tariff schedule** embedded as source data: peak windows Mon–Fri 01:00–04:00 and 06:00–10:00 UTC (from DeepSeek's pricing page); everything else off-peak, including full weekends. User-editable with "Reset to defaults".
- **Background notifications: best effort** — no foreground service; transitions scheduled as close to on-time as Android allows, with possible small delays when the app is fully closed on some devices.
- **Local time shown alongside the UTC clock** on the main screen (labeled with the device timezone), since the user is not on the UTC timezone.
- **Status model**: 4 states (OFF-PEAK green / PEAK-SOON-30 yellow / PEAK-SOON-15 orange / PEAK red); warning thresholds fixed at 30/15 min for v0.1.
- **Offline-only constraint**: no INTERNET permission, no third-party services, no network time sync; relies on the device clock (epoch → UTC, hence DST-immune).
- **Notifications** on every status transition only when the app is not in the foreground; text-only vs sound+text as a user setting; Android 13+ runtime permission required, no nagging if denied.
- **Schedule editor**: per-day peak intervals in UTC, add/remove, validation (end > start, overlap/adjacency merged, midnight-crossing windows entered as two intervals), immediate apply + persistence.
- **Non-goals for v0.1**: no DeepSeek API integration, no widget, no holiday handling, no price display, no localization; future ideas listed in spec §10.
- Open item: min/target Android SDK to be decided at the implementation-plan stage.