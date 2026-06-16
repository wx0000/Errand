# AI history

Archive of the AI working sessions behind this suite: session transcripts and the plans that
drove them. This is the **decision layer** of the deliverable — it shows *how* the suite was
built and *why* defects were ruled BUG/PASS, not just the final files.

`.claude/` is gitignored, so plans only reach the deliverable by being copied here at the end
of each session (see `CLAUDE.md` → End-of-Session Cycle, step 5).

## Naming convention

- **Transcripts:** `YYYY-MM-DD-<phase>-<role>.txt`
  - `role = planning` → plan mode (designing the approach)
  - `role = build` → implementation (executing the approach)
  - example: `2026-06-12-p0-build.txt`
- **Archived plans:** `<phase>-<topic>.md`
  - example: `p0-scaffold-recon.md`

## Index

| Date | Phase | Role | File | Notes |
|------|-------|------|------|-------|
| 2026-06-12 | P0 | build | `2026-06-12-p0-build.txt` | Scaffold built + env diagnosed empty. **Create via `/export` before the commit.** |
| — | P0 | planning | `p0-scaffold.md` | Archived combined Session-0 plan (P0 + P1) — the decision layer. |
| 2026-06-14 | P1 | recon (code) | `2026-06-14-p1-recon-code.txt` | Full recon R1–R5 → SPEC-MATRIX `actual` + candidates C1–C5. **Create via `/export` before the commit.** |
| 2026-06-14 | P1 | adjudication | `2026-06-14-p1-adjudykacja-build.txt` | Human R1–R5 verdicts → SPEC-MATRIX `verdict` + `docs/BUGS.md` (3 root defects BUG-A/B/C). Commit `cd4bf26`. |
| — | P1 | planning | `p1-recon.md` | Archived P1 recon plan — decision layer (recon architecture, STAGE acts, verify-in-build tweaks). |
| 2026-06-15 | P2 | build | `2026-06-15-p2-build.txt` | Plan-mode design + build of subflows + R4/R5 flows; suite 3/3 green; commit `5f61654`. **Create via `/export` before the commit.** |
| — | P2 | planning | `p2-subflows-reg-login.md` | Archived P2 plan — decision layer (subflow boundaries, selector map, `register-user` signature, the env-default reconcile). |
| 2026-06-15 | P3 | build | `2026-06-15-p3-build.txt` | apply (R2 green) + history (R3 C4/BUG-B + C5/BUG-C findings, adjudicated on artifacts) + `register-user` IME-flake fix; suite 3× clean; commits `c8c90a6`, `4664018`. **Create via `/export` before the commit.** |
| — | P3 | planning | `p3-apply-history.md` | Archived P3 plan — decision layer (`apply-to-offer` tap-only, R2 single+multi in one flow, two isolated R3 finding flows, anti-case table). |
| 2026-06-16 | P4 | build | `2026-06-16-p4-build.txt` | R1 offers-list flows 01 (BUG-A) + 02 (BUG-C); suite 8 flows = 4 green + 4 red; flake 3×/0; feat commit `5474ef5` (local, not pushed). **Create via `/export` before the commit.** |
| — | P4 | planning | `p4-r1-offers.md` | Archived P4 plan — decision layer (2 flows no-renumber, BUG-A→BigTech, BUG-B-on-list accepted documented gap, scroll-back-UP framing + masking autoatak). |
| — | P1+P2 | judgment-log | `WORKING-WITH-AI.md` | Running judgment-moments log (P1 + P2; consolidate at closeout). |
