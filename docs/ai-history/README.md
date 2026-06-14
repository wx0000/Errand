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
| — | P1 | judgment-log | `WORKING-WITH-AI.md` | Running judgment-moments log (consolidate at closeout). |
