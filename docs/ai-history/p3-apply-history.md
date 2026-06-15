# P3 — Apply + History (R2 + R3)

## Context

P0–P2 are closed: scaffold + CLAUDE.md (P0), recon + SPEC-MATRIX + adjudication (P1), and the
registration/login suite (P2 — `06`+`07` R4 green, `08` R5 green, 4 subflows + `config.yaml`).
The suite is **3/3 green**. R1 (offers list), R2 (apply), R3 (history) are still unassigned.

P3 delivers **R2** (apply directly from the offers list) and **R3** (application history from the
profile). Per the SPEC-MATRIX + BUGS.md adjudication:
- **R2 = PASS** → expected **green**.
- **R3 = BUG** → two adjudicated manifestations on the history surface:
  - **C4 → BUG-B (history):** a no-salary offer **still renders a salary line** (`Applied job
    salary: not specified` / `—`). Spec: no salary line at all.
  - **C5 → BUG-C (history):** an object-shaped salary **leaks as raw JSON** (`{"min": 10000,
    "max": 15000}`). Spec: a formatted range like every other offer.

R3 is therefore the suite's **first real FINDING**. The flows assert the **SPEC**; a **red** R3
result is the **correct** result. Cardinal rule (binding): we never bend an assertion to a buggy
app. But it cuts both ways — before declaring a finding we **prove the flow correctly drove the
app** (reached history, the right entry) so a red is the app violating spec, not the flow failing.

R3 has a real dependency: history needs prior applies → each R3 flow does
`register → apply (to the right offer) → open-profile → history`.

---

## File layout (final numbering — confirmed with the lead)

Pool `01–05`: **`01`,`02` reserved for R1 offers list (P4)**, **`03` R2**, **`04`,`05` R3**.
Target end-state: 8 flows, contiguous `01–08` (`01`,`02` R1/P4 · `03` R2 · `04`,`05` R3 ·
`06`,`07` R4 · `08` R5).

| New file | Req | Expectation |
|---|---|---|
| `subflows/apply-to-offer.yaml` | — | tap-only helper (param `OFFER`); own-scope gate only |
| `flows/03-apply.yaml` | R2 | single-loc + multi-loc apply in one run · **PASS → green** |
| `flows/04-history-no-salary.yaml` | R3 / C4 / BUG-B | no-salary entry → no salary line · **BUG → red = finding** |
| `flows/05-history-salary-format.yaml` | R3 / C5 / BUG-C | object salary → range, not JSON · **BUG → red = finding** |

Doc edits (build phase): `docs/SPEC-MATRIX.md` (assign R2→`03`, R3→`04`+`05`; fill the anti-case
table) and `PROGRESS.md` (check off P3). Evidence artifacts under **`docs/runs/`**
(`screens/` + `hierarchy/`) — kept separate from P1 `docs/recon/`.

---

## Selector map (anchored to real recon dumps — quoted strings)

All selectors are **regex anchored to the FULL node text (DOTALL)**; tap the **clickable
`Button`**, not the wrapper `View` (CLAUDE.md convention). Company lines are **unique** per offer;
the title `Senior QA` is a substring of `Senior QA Engineer`, so **anchor tiles on company**.

**Offers list** — `docs/recon/hierarchy/02-offers-list-{top,bottom}.txt` (5 offers):

| Offer (title / company) | Locations | Salary (as rendered) | Loc type |
|---|---|---|---|
| Senior QA Engineer / **TechCorp** | WarsawBerlinLondon | `Salary: not specified` / `null` | multi |
| Junior Flutter Developer / **StartupXYZ** | Krakow | `5000-8000 PLN` | single |
| QA Automation Engineer / **BigTech Inc** | WarsawGdansk | `8000-12000 PLN` | multi |
| Mobile QA Tester / **MobileFirst** | Wroclaw | `6000-9000 PLN` | single |
| Senior QA / **QualityAssurance Ltd** | WarsawBerlinLondonParis | `{"min": 10000, "max": 15000}` | multi |

- Per-tile Apply (clickable Button, generic text `Apply`) → disambiguate by company anchor:
  `tapOn: { text: "Apply", below: { text: "${OFFER}" } }` (recon-proven with `.*QualityAssurance.*`).
- List header: `.*Job Offers.*` · Profile entry (used by `open-profile`): `.*Profile button.*`.

**Picker (multi-loc modal)** — `docs/recon/hierarchy/04-apply-multi-picker.txt`:
- Title node: `Select Location for <title>` (e.g. `.*Select Location for QA Automation Engineer.*`).
- Options (clickable Buttons): `.*Location option: Warsaw.*`, `.*Location option: Gdansk.*`,
  `.*Location option: London.*`.

**Snackbar (transient — caught in-flow, not in a parked dump)** — token `.*Applied to.*`;
full content is one node `Applied to <title> - <location>` (SPEC-MATRIX R2). Assert title **AND**
location together via `extendedWaitUntil: visible:` on the content regex; `takeScreenshot` = artifact.

**History** — `docs/recon/hierarchy/09-history.txt` (reached via `open-profile`):
- Section gate: `.*Application History.*` · entry lines: `Applied job title|company|location|salary: <val>`.
- C4 node (no-salary): `Applied job salary: not specified` / `—`.
- C5 node (object salary): `Applied job salary: {"min": 10000, "max": 15000}`.

---

## Implementation detail (which assertions = SPEC)

### `subflows/apply-to-offer.yaml` — tap-only (param `OFFER`)
Single responsibility = tap the Apply Button of the target tile. Own-scope gate only; the
outcome (snackbar for single-loc, picker for multi-loc) is the **caller's** assertion territory —
mirrors `register-user` (attempt, no outcome). **NO `env:` defaults** (Maestro 2.6.1 precedence
bug; caller passes `OFFER`, a missing one then errors loudly).

```
- assertVisible: ".*Job Offers.*"          # own-scope gate
- scrollUntilVisible: { element: { text: "${OFFER}" }, direction: DOWN }
- tapOn: { text: "Apply", below: { text: "${OFFER}" } }   # this tile's clickable Apply
```

### `flows/03-apply.yaml` — R2 (PASS, expected green)
`launchApp clearState` → unique email (`evalScript` + `Date.now()`) → `register-user` →
`extendedWaitUntil .*Job Offers.*`. Then **two cases of the same requirement** in one run (both
PASS, so no assertion masks another):
- **Single-loc** (Junior / StartupXYZ): `apply-to-offer OFFER=.*StartupXYZ.*` → **SPEC:**
  `extendedWaitUntil visible ".*Applied to.*Junior Flutter Developer.*Krakow.*"` (title **AND**
  location) → `takeScreenshot docs/runs/screens/03-apply-single-snackbar` →
  `extendedWaitUntil notVisible ".*Applied to.*"` (dismiss before next apply).
- **Multi-loc** (QA Automation / BigTech): `apply-to-offer OFFER=.*BigTech.*` → **SPEC:**
  `assertVisible ".*Select Location for QA Automation Engineer.*"` (picker opens first) →
  `tapOn ".*Location option: Gdansk.*"` → **SPEC:**
  `extendedWaitUntil visible ".*Applied to.*QA Automation Engineer.*Gdansk.*"` (title **AND**
  chosen location) → `takeScreenshot docs/runs/screens/03-apply-multi-snackbar`.

### `flows/04-history-no-salary.yaml` — R3 / C4 / BUG-B (BUG, expected red)
`register → apply-to-offer OFFER=.*TechCorp.*` (Senior QA Engineer; multi-loc → `assertVisible
".*Select Location for Senior QA Engineer.*"` → `tapOn ".*Location option: Warsaw.*"`) →
snackbar visible/notVisible → `open-profile` → `extendedWaitUntil ".*Application History.*"`.
- **Mechanic proof (green):** `assertVisible` title `.*Applied job title: Senior QA Engineer.*`,
  company `.*Applied job company: TechCorp.*`, location `.*Applied job location: Warsaw.*` →
  `takeScreenshot docs/runs/screens/04-history-no-salary`.
- **SPEC assertion (the finding):** `assertNotVisible: ".*Applied job salary.*"` — a no-salary
  entry must have **no salary line at all** (single-entry history → no legit salary line to catch).
  Buggy renders `not specified`/`—` → this **fails → RED = C4**. We do **not** assert the
  placeholder present (that would rubber-stamp BUG-B).

### `flows/05-history-salary-format.yaml` — R3 / C5 / BUG-C (BUG, expected red)
`register → apply-to-offer OFFER=.*QualityAssurance.*` (Senior QA / QualityAssurance Ltd;
multi-loc → `assertVisible ".*Select Location for Senior QA.*"` → `tapOn ".*Location option:
London.*"`) → snackbar visible/notVisible → `open-profile` → `extendedWaitUntil ".*Application
History.*"`.
- **Mechanic proof (green):** `assertVisible` company `.*Applied job company: QualityAssurance
  Ltd.*` + location `.*Applied job location: London.*` (company is unique; avoids the `Senior QA`
  substring trap) → `takeScreenshot docs/runs/screens/05-history-salary-format`.
- **SPEC assertion (the finding):** `assertVisible: ".*Applied job salary: 10000-15000.*"` —
  salary shown as a formatted **range** (the known data is min 10000 / max 15000, PLN-agnostic
  regex). Buggy leaks raw JSON → the range text is absent → this **fails → RED = C5**. We do
  **not** assert the JSON visible (that would rubber-stamp BUG-C).

---

## Anti-case table (drafted — for the lead to ratify at approval)

SPEC-MATRIX flags anti-cases as **the human's judgment**. These rows replace the format-example
placeholders (lines 96–101) and are derived directly from the adjudicated bugs + the cardinal
rule. They are presented for **ratification**, not authored unilaterally.

| Anti-case (do NOT assert) | Why it would over-interpret | Related Req |
|---|---|---|
| snackbar shows only the title (omit the applied location) | R2 requires title **AND** location; title-only locks in an under-spec snackbar | R2 |
| a no-salary history entry renders a salary line/placeholder (`—` / "not specified") | spec omits salary for no-salary offers; asserting the placeholder present rubber-stamps **BUG-B** | R3 |
| an object salary asserted as raw JSON `{"min": …}` | spec requires a formatted range; asserting the JSON rubber-stamps **BUG-C** | R3 |
| application history / session **survives an app restart** | spec is **silent** on restart; recon (screens 12/13) shows it survives, but asserting persistence over-interprets the app | R3 (persistence) |

The R1/BUG-A anti-case (multi-location concatenation asserted as one token) lands with P4's R1 flows.

---

## Build-verify items (verify against the real emulator BEFORE/while implementing)

The build does **not** execute this plan blindly — if reality contradicts it, **STOP and reconcile**.

1. **Preflight:** `adb devices` (emulator up), `maestro` on PATH, app installed
   (`adb shell pm list packages | grep buggy`).
2. **Snackbar content regex catchability (highest risk):** confirm `extendedWaitUntil` catches the
   **full** `.*Applied to.*<title>.*<loc>.*` (recon only proved the bare `.*Applied to.*` token).
   The snackbar text is a single node, so the stricter regex should match the same node — but if it
   does **not** catch, **STOP and reconcile** (do not silently weaken to title-only — that is the
   R2 anti-case).
3. **Apply disambiguation:** confirm `tapOn { text: Apply, below: company }` taps the intended
   tile (recon-proven for `QualityAssurance`); confirm single-loc Junior goes **straight to the
   snackbar** (no picker), multi-loc opens the picker.
4. **R2 must be green.** If the build shows an **R2 defect** (e.g. snackbar title-only, no
   location), the first red assertion **masks the multi case** in the same flow → **STOP, report,
   wait for the lead's decision** (split R2, or bundle R3). **Do not route around this.**
5. **R3 findings reproduce:** confirm the no-salary entry still renders `not specified`/`—` (C4)
   and the object salary still leaks `{"min": …}` (C5). The mechanic-proof asserts (title/company/
   location) must pass **green first** — proving the red is the app, not the flow.

---

## Run / report / STOP (End-of-Session is NOT now)

1. Build subflow + `03` → `maestro test flows/03-apply.yaml` → expect **green** (else item 4 above).
2. Build `04`,`05` → run **each individually** → expect **red at the salary assertion**, green
   mechanic-proof first. Immediately after each red run, **park-then-dump** (nothing in between):
   `maestro hierarchy > docs/runs/hierarchy/04|05-…txt` while the app is parked on history.
3. Full suite `maestro test .` → report the matrix: `03`/`06`/`07`/`08` green, `04`/`05` **red =
   findings**. Paste the real Maestro output (no fabrication).
4. Update SPEC-MATRIX (flow assignments + anti-case table) + PROGRESS.
5. **STOP** with the red R3 artifacts (screenshots + hierarchy dumps) + a report → **wait for the
   lead's adjudication**. **Do NOT commit the findings.** Linking the red artifacts to `BUGS.md`
   is **P5**, not now. The full End-of-Session cycle (commit/export/archive) is deferred.

---

## Decisions

**Resolved with the lead (this session):**
- R3 = **two isolated flows** (`04` C4, `05` C5) — Maestro aborts on the first failed assertion,
  so one combined flow would mask the second finding.
- Numbering: `03` R2, `04`+`05` R3 (`01`,`02` reserved for R1/P4).
- R2 = **one flow** (`03`) covering single + multi — both PASS, same requirement, nothing masks.
- Build STOP-condition if R2 shows a defect (don't route around — item 4).
- `apply-to-offer` = **tap-only** (param `OFFER`); picker + snackbar handled in the flow.

**My recommendations (minor — ratify at approval):**
- Evidence dir = `docs/runs/{screens,hierarchy}/` (separate from P1 `docs/recon/`).
- Tile anchor = **company** line (unique; avoids the `Senior QA` ⊂ `Senior QA Engineer` trap).
- C4 assertion = `assertNotVisible ".*Applied job salary.*"` (no salary line at all; safe because
  the isolated flow's history has exactly one entry).
- C5 assertion = `assertVisible ".*Applied job salary: 10000-15000.*"` (positive spec: a range;
  PLN-agnostic), not `assertNotVisible` the JSON.

## Out of scope (P3)
R1 offers list (P4). Rejected extras (tags/selective-run, CI, iOS, perf, visual) → README future
work, never suite code. End-of-Session cycle + commit + BUGS.md linking → later (P5 for linking).
