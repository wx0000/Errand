# CLAUDE.md — Buggy Maestro E2E suite

> **Read this file at the start of every session.**
> Conversation with the CLI is in Polish; **every file in this repo is written in English.**

---

## What we are building

A **Maestro E2E test suite** for the Android app **Buggy**, asserting the behavior the
**task specification** requires. The deliverable is ≥5 E2E tests plus a README that brings
the suite up from a clean clone on the reviewer's machine.

The reviewer's environment equals the dev environment: **macOS, MacBook Air M2, an Android
ARM64 emulator driven over `adb`.**

## Project facts

| | |
|---|---|
| App | **Buggy** — Flutter **release** build, `arm64-v8a` |
| appId | `com.example.buggy` |
| Network | **Offline** — the app has **no `INTERNET` permission**; do not write tests that assume network |
| Tooling | **Maestro** (flows in YAML) |
| Device | Android **ARM64 emulator** via `adb` |
| Repo | **Public** — see Hygiene below |

---

## Cardinal rule (binding)

**Tests assert the SPECIFICATION, never the app's behavior.**

The app almost certainly contains **intentional defects** versus the spec. A test that fails
because the app violates the spec is doing its job — that is a *finding*, not a test bug.

> **"Adjust the assertion so it passes" against behavior that contradicts the spec is a
> disqualification.** If that move ever surfaces in reasoning — even as a tempting shortcut —
> **name it explicitly** and reject it. We never bend an assertion to match a buggy app.

This session (and recon generally) produces **material**; the human issues the **BUG/PASS
verdicts** from the screenshots. Claude does not rule defects.

---

## SPEC MATRIX

**SPEC MATRIX = binding ground-truth, lives in `docs/SPEC-MATRIX.md`** (single source: R1–R5
expected-per-spec, recon `actual`, bug candidates, anti-case placeholders; `verdict` column is
the human's, never Claude's). Do not keep a second copy of the skeleton here.

---

## Plan-bridge (how we work)

- **One terminal, plan mode on a toggle.** When thinking is needed before writing (recon →
  SPEC MATRIX), switch plan mode ON in the same window, design the plan, save it to
  `.claude/plans/<phase>-<topic>.md`, switch plan mode OFF, then build from the file.
- **The plan bridge is the file** in `.claude/plans/`. Plan mode is a *mode*, not a separate
  terminal.
- **Judgment stays in the main window** (stress-testing the plan, the defect-or-not call) —
  the transcript is part of the deliverable. Mechanics may be delegated to a subagent.
- **Build verifies the plan against the real emulator before implementing** — it does not
  execute the plan blindly. If reality contradicts the plan, stop and reconcile.

## Self-Validation Protocol (apply before showing anything)

1. **Every bug candidate** names: which requirement it asserts, the expected-per-spec, and the
   **real evidence** — a string quoted from the hierarchy dump, or a screenshot filename.
   Never a *predicted* result.
2. **Paste real run outputs** (maestro, adb). Never fabricate command output.
3. **Flag your own flailing.** If bash/maestro has not returned a result yet, do **not** narrate
   a result you don't have — stop, wait for the real output, then conclude.
4. **Anti-sycophancy.** When a move is questioned, defend it with evidence or concede with a
   stated reason. No reflexive agreement.

## End-of-Session Cycle (run BEFORE closing a session, in order)

If a step does not apply, **say which and why — never skip silently.**

1. **Suite green/red** — run the suite, report the result.
2. **Update CLAUDE.md** — Current Status (phase, next task, R1–R5 coverage) + a Session Log entry
   (what was done, decisions, judgment moments).
3. **Update PROGRESS.md** — check off what's done, note any drift if the plan changed.
4. **`/export`** this session to `docs/ai-history/` as `YYYY-MM-DD-<phase>-<role>.txt`.
5. **Archive the plan — UNCONDITIONAL.** Every plan created this session is copied from
   `.claude/plans/` to `docs/ai-history/` as `<phase>-<topic>.md` before the session closes —
   **always, even a minor plan.** Reason: `.claude/` is gitignored, and the plans are the
   **decision layer** of the delivery docs that the build transcript alone cannot reconstruct.
6. **Update the index** in `docs/ai-history/README.md` with the new files.
7. **Local commit** — conventional commits, one commit = one logical unit (scaffold separate
   from recon), no squash; history tells the process. **Push is the human's call — do not
   commit without explicit consent.**
   **Never add `Co-Authored-By` or "Generated with Claude Code" trailers to commits or PRs.**
   (Attribution is disabled in `.claude/settings.local.json`; this line is the backup, since the
   setting is intermittently ignored — known bug.)
8. **Handoff prompt** for the next session (what's done, what's next, what to watch, which file
   to read first) + a recommended effort with a one-sentence justification.

## Hygiene

- **`buggy.apk` and the task PDF are in `.gitignore` from the first commit** and must never
  enter the repo or its history (public repo, company-owned artifacts).
- These are the **only** forbidden files. Recon artifacts (hierarchy dumps, screenshots) are
  committed as evidence for the human's verdicts.
- `.claude/` is gitignored; plans reach the deliverable only via the archival step above.

---

## Current Status

- **Phase:** **P0 (scaffold) — DONE.**
- **Next task:** **P1 recon — BLOCKED** on two prerequisites: (1) toolchain install
  (modern JDK + Maestro + Android SDK/adb + an `arm64-v8a` emulator + `adb install buggy.apk`),
  and (2) the human pasting requirements **R1–R5**.
- **R1–R5 coverage:** none — requirements not yet provided; SPEC MATRIX rows are placeholders.

## Session Log

> Newest entries at the top. Each entry: date, phase, what was done, decisions, judgment moments.

### 2026-06-12 — P0 — build

- **Done:** scaffold built (`.gitignore`, `CLAUDE.md`, `PROGRESS.md`,
  `docs/ai-history/README.md`). Environment verified — **empty**: no Maestro, no adb/Android
  SDK (`ANDROID_HOME` unset), JDK present but **Java 9.0.4 (EOL)**. Install commands handed to
  the human; **I did not install.** Recon **not** started — double-blocked (toolchain + R1–R5).
- **Decisions:** Polish conversation / English files. Recon artifacts are committed as evidence;
  only `buggy.apk` + the task PDF are excluded. `.claude/` gitignored. Plan archival is an
  unconditional end-of-session rule.
- **Judgment moments:**
  1. Caught a prompt-vs-reality mismatch — the dir was **not** "empty + .gitignore"; it had a
     tracked `README.md`, **no** `.gitignore`, and a **live public remote** (`origin/main`).
  2. Made `.gitignore` the **first** scaffold action precisely because that public remote was
     already live (APK/PDF leak risk is immediate, not hypothetical).
  3. Distinguished **PATH-gap vs not-installed** — probed the default install locations
     (`~/.maestro/bin`, `~/Library/Android/sdk`, Homebrew) before telling the human to install;
     confirmed genuinely absent, not merely off-PATH.

<!-- entries appended at end of each session -->
