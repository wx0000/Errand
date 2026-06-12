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
| — | P0 | planning | `p0-scaffold.md` | Archived plan — the decision layer behind P0. |
