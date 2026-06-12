# Plan — JJIT recruitment task: Buggy Maestro E2E suite (Session 0: P0 scaffold + P1 recon)

## Context

Recruitment task for JJIT: deliver ≥5 Maestro E2E tests for the Android app **Buggy**
(Flutter release, arm64-v8a, offline, no INTERNET permission, appId `com.example.buggy`),
plus a README that brings the repo up from a clean clone. The repo is **public**, so
`buggy.apk` and the task PDF must never enter the repo or its history.

Cardinal rule for the whole project: **tests assert the SPECIFICATION, never the app's
behavior.** The app almost certainly contains intentional defects vs spec. This session
does **not** issue BUG/PASS verdicts — it produces the material (recon evidence) on which
the human issues verdicts from screenshots.

Scope of Session 0 = **P0 (foundation/scaffold) + P1 (recon → SPEC MATRIX material)**.
No test flows/subflows yet (those start P2).

## Starting state (verified against the real repo, not the prompt's assumptions)

- Working dir contains only `README.md` (content: `# Errand`) and `.git`. **There is NO
  `.gitignore`** — contrary to the prompt's "pusty plus .gitignore". This means the
  hygiene rule is currently UNMET.
- A **public remote already exists** (`origin/main`, tracking `README.md`). APK/PDF leak
  risk is live now → `.gitignore` must be the first scaffold action, before any APK/PDF
  ever lands in the directory.
- Nothing else to explore (empty project) → no Explore/Plan subagents used; judgment work
  stays in the main window because the transcript is part of the deliverable.

## Decisions

- **Language:** conversation with Claude CLI in Polish; **all project files in English**
  (CLAUDE.md, PROGRESS.md, README, BUGS.md, TEST_PLAN, docs, future flow names).
- **Env verification timing:** run `maestro`/`adb` checks only in step 2 (post-acceptance),
  so the real outputs land in the build transcript. Not run during planning.
- **Subagents:** none. Empty repo + judgment-in-main-window rule.
- **Repo footprint:** public repo confirmed. Recon artifacts (screenshots, hierarchy dumps)
  are committed as evidence; ONLY `buggy.apk` + task PDF are excluded. `.claude/` is gitignored.
- **Plan archival** is a project-wide UNCONDITIONAL rule (CLAUDE.md End-of-Session step 5);
  this session's final plan is archived to `docs/ai-history/p0-scaffold.md` at close (plans use
  `<phase>-<topic>.md`, no date; the date appears only on `.txt` transcripts).

---

## P0 — Foundation / scaffold (after plan acceptance)

Create, in this order:

### 1. `.gitignore` (FIRST — before anything else)
Hard-exclude from repo AND history (company-owned artifacts) — per spec the ONLY forbidden files:
- `buggy.apk`, `*.apk`, `*.apks`, `*.aab` (the build artifact must never be committed)
- the task PDF (e.g. `*.pdf` and/or the exact task filename) — proprietary JJIT material
Also ignore:
- `.claude/` — session-local Claude state; plans live here and are archived to `docs/ai-history/`
  instead (see End-of-Session step 5)
- standard noise: `.DS_Store`, Maestro/IDE temp output
**Recon artifacts (hierarchy dumps, screenshots) DO go into the repo as evidence** — decided,
no re-confirmation at recon-commit time. Only `buggy.apk` and the PDF are excluded.

### 2. `CLAUDE.md` — built FROM SCRATCH per spec (describes Buggy, not any structural inspiration)
Sections:
- **Header line:** "Read this file at the start of every session."
- **What we are building:** a Maestro E2E suite for Buggy per spec.
- **Project facts:** app = Flutter release, arm64-v8a, offline, no INTERNET permission,
  appId `com.example.buggy`. Env = macOS, MacBook Air M2, Android ARM64 emulator via adb.
  Tooling = Maestro. Dev env == reviewer env.
- **Cardinal rule (binding):** tests assert spec, not the app. "Adjust the assertion so it
  passes" against behavior that contradicts spec = **disqualification** — name it explicitly
  if it ever surfaces in reasoning.
- **SPEC MATRIX** (replaces any "golden dataset"):
  - Main table R1–R5, columns: `requirement | flow | expected per spec | actual (from recon)
    | verdict PASS/BUG`. The **verdict column stays EMPTY** until the human review.
  - Second half — **anti-cases**: what a test must NOT assert because it would be
    over-interpreting the app. This session these are **FORMAT PLACEHOLDERS only** (column/
    shape example taken from the prompt — NOT real Buggy findings): snackbar showing only the
    title instead of title+location; a placeholder the spec forbids; a validation error
    rendered below the input instead of above. **Real anti-cases are filled only after R1–R5
    and recon** — do not treat these as ready app cases.
- **Plan-bridge:** one terminal + plan mode on a toggle; the plan bridge is a file in
  `.claude/plans/`. Build verifies the plan against the real emulator before implementing —
  it does not execute blindly.
- **Self-Validation Protocol** (verbatim intent): (1) every bug candidate names which
  requirement it asserts, expected-per-spec, and real evidence (hierarchy string or
  screenshot name) — never predicted; (2) paste real run outputs (maestro, adb), never
  fabricated; (3) flag own flailing — if bash/maestro hasn't returned, do not narrate a
  result you don't have; stop, wait for the real output, then conclude; (4) anti-sycophancy
  — when a move is questioned, defend it with evidence or concede with a stated reason.
- **End-of-Session Cycle** (the 8 steps, verbatim intent): green/red suite; update CLAUDE.md
  (Current Status + Session Log); update PROGRESS.md; `/export` to docs/ai-history;
  **archive plan (UNCONDITIONAL):** every plan created in the session is copied from
  `.claude/plans/` to `docs/ai-history/` as `<phase>-<topic>.md` before the session closes —
  always, even a minor plan, because `.claude/` is gitignored and the plans are the decision
  layer of the delivery docs that the build transcript alone cannot reconstruct; update
  ai-history index; local commit (conventional, one logical unit each, no squash, push decided
  by human, no commit without consent); handoff prompt + recommended effort.
- **Hygiene:** `buggy.apk` and the PDF in `.gitignore` from the first commit.
- **Current Status:** phase, next task, R1–R5 coverage (filled at end of session).
- **Session Log:** running log (what done, decisions, judgment moments).

### 3. `PROGRESS.md`
Phases: P0 scaffold+CLAUDE.md | P1 recon+SPEC MATRIX | P2 subflows + register/login |
P3 apply + history | P4 offers list | P5 stabilization 3× + BUGS.md + TEST_PLAN + README +
fresh-clone. Plus a **Session Log** format (date, phase, done, drift).

### 4. `docs/ai-history/README.md`
Transcript naming convention: `YYYY-MM-DD-<phase>-<role>.txt`
(role: `planning` = plan mode, `build` = implementation) + an empty index table.

### P0 — Environment verification (step 2, post-acceptance; do NOT install, only check)
Run and paste **real** outputs:
- `maestro --version`
- `adb devices` (expect an authorized ARM64 emulator)
- `adb shell pm list packages | grep buggy`
If anything is missing: give the exact install/start command and **STOP** — the human installs.

---

## P1 — Recon (only AFTER the human pastes requirements R1–R5)

- `recon.yaml`: launch Buggy, traverse the key screens.
- `maestro hierarchy` dump per screen → one file per screen (under e.g. `docs/recon/`).
- Screenshot per screen → one file per screen.
- Fill the **actual** column of the SPEC MATRIX from what is really visible.
- Note **bug candidates**: per candidate → which requirement (R#), expected per spec, what is
  observed (quote of the string/hierarchy + screenshot filename). **No verdict** — this is
  material for the human's ruling.

---

## Out of scope this session (guardrails)

- No test flows/subflows (those are P2). Recon + SPEC MATRIX material only.
- No BUG/PASS verdicts — the human rules from screenshots.
- No "adjust the assertion to pass" — name that anti-pattern explicitly if it appears.
- No extras (CI, tags, junit). Baseline only.
- No copying files from other projects — keep the dir clean.
- No committing without explicit human consent (push is the human's call).

## Commits (end of session, only with consent)
Two separate logical commits, conventional, no squash: (a) scaffold, (b) recon. History
tells the process. Ask before running `git commit`.

## Verification (how we know this session succeeded)
- `.gitignore` exists and ignores `*.apk`/PDF **before** any such file is present (`git
  check-ignore buggy.apk` would confirm post-creation).
- CLAUDE.md contains every required section incl. the empty-verdict SPEC MATRIX and the
  anti-cases half.
- Env outputs are real and pasted; if a tool is missing, an exact command is given and we stop.
- After R1–R5: recon.yaml runs on the real emulator; per-screen hierarchy dumps + screenshots
  exist; the `actual` column is filled from real evidence; bug candidates listed with R#,
  expected, evidence quote + screenshot name, and NO verdicts.
