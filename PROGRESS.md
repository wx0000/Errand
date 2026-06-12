# PROGRESS — Buggy Maestro E2E suite

Phase tracker. Tests assert the **spec**, not the app (see `CLAUDE.md` → Cardinal rule).
Update at the end of every session: check off what's done, note drift if the plan changed.

## Phases

- [x] **P0 — Scaffold + CLAUDE.md**
  - [x] `.gitignore` (buggy.apk + PDF excluded from the first commit; `.claude/` ignored)
  - [x] `CLAUDE.md` built from scratch (SPEC MATRIX, cardinal rule, protocols, cycle)
  - [x] `PROGRESS.md`
  - [x] `docs/ai-history/README.md` (transcript naming convention + index)
  - [x] Environment verification — **run; env empty** (no maestro/adb/SDK; JDK 9.0.4 EOL); exact install commands handed to human

- [ ] **P1 — Recon + SPEC MATRIX** — **BLOCKED** (waits on toolchain install **and** the human pasting R1–R5)
  - [ ] `recon.yaml` — launch Buggy, traverse key screens
  - [ ] Per-screen Maestro hierarchy dumps → files
  - [ ] Per-screen screenshots → files
  - [ ] Fill the `actual` column of the SPEC MATRIX from real evidence
  - [ ] List bug **candidates** (R#, expected per spec, evidence quote + screenshot) — **no verdicts**

- [ ] **P2 — Subflows + registration / login**
  - [ ] Reusable subflows (launch, auth helpers, …)
  - [ ] Registration flow
  - [ ] Login flow

- [ ] **P3 — Applying + history**
  - [ ] Apply-to-offer flow
  - [ ] History flow

- [ ] **P4 — Offers list**
  - [ ] Offers list flow(s)

- [ ] **P5 — Stabilization + delivery**
  - [ ] Run the full suite 3× (stability)
  - [ ] `BUGS.md` (human verdicts → documented defects)
  - [ ] `TEST_PLAN`
  - [ ] `README` (clean-clone bring-up)
  - [ ] Fresh-clone verification

## Session Log

> Newest entries at the top. Format:
> `### YYYY-MM-DD — <phase> — <role>` then bullets: **Done**, **Decisions**, **Drift**.

### 2026-06-12 — P0 — build

- **Done:** P0 scaffold complete; env verified empty (install handed to human).
- **Decisions:** scope split — P0 is its own transcript; recon deferred to a fresh session.
- **Drift:** none vs plan. P1 blocked on toolchain install + R1–R5.

<!-- entries appended at end of each session -->
