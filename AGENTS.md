# AGENTS.md — Project Working Rules (v0.1)

Contract between the user and the agent (Reasonix) for this project. This file and `spec.md` are authoritative. Any change to this file or `spec.md` requires the user's explicit permission and is logged in `PROJECT_LOG.md`.

## Project
A very simple, very limited Android app for the user's personal use. Purpose, features and scope will be defined in `spec.md` (next phase).

## 1. Repository & commits
- All work lives in this git repository. The repo must be clonable elsewhere and development resumable from a fresh clone.
- Commit after every completed step (setup, spec, each milestone). Never leave uncommitted work at the end of a session.
- Work directly on `main` with direct commits (single-developer personal project); no feature branches unless the user asks.
- Commit messages: short, imperative, specific about what changed.
- Stage explicit file paths only — never `git add .` or `git add -A`.
- Never stage or commit `my_considerations.txt`.

## 2. my_considerations.txt
- The user's personal scratch file. For the user only.
- Never read, never edit, never stage, never commit, never mention it.
- User changes to it between the agent's commits are ignored silently.
- It stays tracked in git (the user commits it); do not add it to `.gitignore`.

## 3. .gitignore
- Maintained by the agent as needed. Ignore anything that gets rebuilt or redownloaded: build outputs (`build/`, `.gradle/`, `.kotlin/`), local Android/Gradle config (`local.properties`), IDE folders (`.idea/`, `*.iml`), OS junk, and `.reasonix/` (local Reasonix session state).
- Commit everything essential to rebuild/redevelop: source, configs, docs (`AGENTS.md`, `spec.md`, `PROJECT_LOG.md`, `.gitignore`), and the Gradle wrapper including `gradle-wrapper.jar`.

## 4. spec.md (project contract)
- Authoritative source of truth for what the project and app are: goal, features, non-goals, constraints, acceptance criteria.
- Versioned at the top, starting at 0.1; version increments on every change.
- Never modified autonomously. If a change seems needed, ask the user for explicit permission first.
- Every change documented in `PROJECT_LOG.md` (timestamp + description).

## 5. AGENTS.md (these rules)
- Same protection as `spec.md`: changes require explicit permission; every change is logged in `PROJECT_LOG.md`; versioned (this file: v0.1).

## 6. PROJECT_LOG.md
- Append-only change log, never rewritten in place.
- Records: all changes to `spec.md` and `AGENTS.md`, plus significant project decisions (architecture, dependencies, why-choices), each with a timestamp and description.

## 7. Decision rule
- Consequential choices (spec/AGENTS changes, architecture, dependencies) are submitted to the user for approval before being applied.
- Reversible, low-risk choices default to the sensible option.

## Notes
- (empty stub — build/test commands and Android tooling details will be added here when the app is scaffolded)
