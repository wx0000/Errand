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
5. **Explain before every approval gate.** Before every action that triggers a user approval
   prompt — file edit, bash command, file read, anything — state in 1–3 plain-language sentences
   what it does and why, in the prose immediately before it. No bare action at an approval gate.

## Maestro / selector conventions (P1 recon — load-bearing for P2)

Learned from real recon against Buggy; apply when writing the P2 flows.

- **Selectors are regex anchored to the FULL node text (DOTALL).** Flutter exposes text as
  `accessibilityText` / `hintText` in a `"label\nvalue"` shape, so a bare substring does **not**
  match — wrap tokens (`.*token.*`) or use the exact full string.
- **Tap the clickable `Button`, not the wrapper `View`** — the wrapper is often
  `clickable=false` and the tap fails.
- **Pass `STAGE` only via `-e STAGE=<act>`.** Do **not** add `env: STAGE: …` in the flow header:
  in Maestro 2.6.1 the flow `env` default overrides `-e` and silently pins every run to the default.
- **Snackbar = transient, absent from the a11y hierarchy.** Catch it in-flow with a screenshot +
  `extendedWaitUntil` on its token (`.*Applied to.*`); the **screenshot is the evidence**.
- **Picker = modal, parkable** → dump its hierarchy out-of-band (`maestro hierarchy`).
- **`maestro hierarchy`** (no flag) = JSON; **`--compact`** = CSV.

The full per-screen selector map lives in the comments of `recon.yaml` (repo root).

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
- **Scratch artifacts are NEVER committed.** Throwaway/scratch artifacts (smoke tests,
  `_`-prefixed scratch, temp files, `/tmp` dumps) never enter a commit. Before **any** commit,
  `git status` shows only real deliverables. At End-of-Session, confirm no scratch file is staged
  or tracked — only real recon evidence under `docs/recon/` is committed.

---

## Current Status

- **Phase:** **P1 — DONE** (recon + adjudication). SPEC MATRIX `verdict` filled from the human's
  ruling; `docs/BUGS.md` records **3 root defects** (BUG-A/B/C — 5 manifestations across R1 list +
  R3 history). Adjudication committed `cd4bf26`.
- **Next task:** **P2 — subflows + registration/login.** First real suite flows (reusable
  subflows: launch + auth helpers → registration → login) against the live emulator, applying the
  Maestro/selector conventions above. Tests assert the SPEC, never the app.
- **R1–R5 coverage:** all 5 adjudicated — **R1 BUG** (C1→BUG-A, C2→BUG-B, C3→BUG-C),
  **R2 PASS**, **R3 BUG** (C4→BUG-B, C5→BUG-C), **R4 PASS**, **R5 PASS**. Per-requirement `flow`
  assignment still _TBD — P2_.

## Session Log

> Newest entries at the top. Each entry: date, phase, what was done, decisions, judgment moments.

### 2026-06-14 — P1 — adjudication

- **Done:** human issued the R1–R5 verdicts → recorded in `docs/SPEC-MATRIX.md` (`verdict` column;
  `actual` left byte-untouched) and created `docs/BUGS.md` — **3 root defects**: BUG-A (multi-loc
  no separator, R1), BUG-B (missing-salary placeholder, R1 list + R3 history), BUG-C (object salary
  as raw JSON, R1 list + R3 history); 5 manifestations on 2 surfaces. C1–C5 mapped to bug-ids.
  Committed `cd4bf26`.
- **Decisions:** `docs/BUGS.md` written in **English** (repo convention) though the human drafted
  it in Polish — confirmed with the human before writing. Verdicts are the **human's**; Claude only
  recorded them (verdict-column rule re-stated to "records, never authors").
- **Judgment moments:**
  1. Caught a prompt-internal conflict — verbatim Polish "exact content" vs. the binding "every
     file in English" rule + "keep repo style" — surfaced it and asked instead of silently choosing.
  2. Self-audited the diff before commit: confirmed the `actual` column is byte-identical (only the
     `verdict` cell appended), so no spec-evidence was altered.
  3. Found `Current Status` stale (still "P0 done / P1 recon blocked") and the recon session left
     no CLAUDE.md Session Log entry — refreshed Current Status; flagged the recon-entry gap to the
     human rather than fabricating it.

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
