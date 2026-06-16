# P4 — R1 offers list (flows 01 + 02)

## Context

R1 ("offers list — tile data") is the only requirement still without executable flows. The human's
verdict in `docs/SPEC-MATRIX.md` is **BUG**: the offers list is the *list surface* of the same
formatting cluster already found in history (R3). Three manifestations on the list:

- **BUG-A (C1):** multi-location values concatenated with **no separator** (`WarsawGdansk`,
  `WarsawBerlinLondon`, `WarsawBerlinLondonParis`). Spec = **comma-separated**.
- **BUG-B (C2):** a no-salary offer renders a salary element (`null` / "Salary: not specified").
  Spec = **no salary element at all** when salary is absent.
- **BUG-C (C3):** an object-shaped salary leaks as **raw JSON** `{"min": 10000, "max": 15000}`.
  Spec = a **formatted range** like every other offer (`10000-15000 PLN`).

The tests assert the **SPECIFICATION**. A RED here is the **correct result** = the finding; we never
bend an assertion to the buggy output (cardinal rule). Outcome: 2 new flows, both expected RED.

### Locked decisions (lead, this session)

- **2 flows in the 01-08 scheme, no renumbering.** 01 = BUG-A, 02 = BUG-C.
- **BUG-B on the LIST surface = accepted, documented gap** (not a 3rd flow). Root defect is already
  RED in history (`04-history-no-salary`); the list manifestation is documented in the SPEC-MATRIX
  anti-case ("a salary placeholder for a no-salary offer — `null` on the list, `—` in history").
  Rationale: Maestro aborts on the first failed assertion, so BUG-B-list cannot share 01/02 without
  masking; covering it would need a 3rd flow + reopening the count decision. Root defect is proven →
  accept.
- **BUG-A target tile = BigTech Inc (`WarsawGdansk`)** — the only tile carrying *just* BUG-A (clean
  salary `8000-12000 PLN`), so the screenshot is a single, unambiguous finding for the human's
  verdict; simplest spec string `Warsaw, Gdansk`. Cost: one `scrollUntilVisible direction: UP`
  before the red (the tile is mid-list; the 5-offer proof scrolls past it to the bottom).

## Ground-truth (verified verbatim from recon dumps, 2026-06-14)

Source: `docs/recon/hierarchy/02-offers-list-top.txt` + `02-offers-list-bottom.txt` (offline release
build → static content; re-confirm with a fresh `maestro hierarchy` at build start).

| # | Title | Company (anchor) | Locations node | Salary node | Defect |
|---|---|---|---|---|---|
| 1 | Senior QA Engineer | `TechCorp` | `WarsawBerlinLondon` | `Salary: not specified` / `null` | BUG-A **+** BUG-B |
| 2 | Junior Flutter Developer | `StartupXYZ` | `Krakow` | `Salary: 5000-8000 PLN` | clean |
| 3 | QA Automation Engineer | `BigTech Inc` | `WarsawGdansk` | `Salary: 8000-12000 PLN` | **BUG-A only** |
| 4 | Mobile QA Tester | `MobileFirst` | `Wroclaw` | `Salary: 6000-9000 PLN` | clean |
| 5 | Senior QA | `QualityAssurance Ltd` | `WarsawBerlinLondonParis` | `Salary: {"min": 10000, "max": 15000}` | BUG-A **+** BUG-C |

**Anchor on COMPANY, never on title** — "Senior QA" (offer 5) is a substring of "Senior QA Engineer"
(offer 1). Company names are all unique substrings (`TechCorp`, `StartupXYZ`, `BigTech`,
`MobileFirst`, `QualityAssurance`). (Same collision the lead highlighted in `04-history-no-salary`.)

## Conventions reused (no new subflow)

- Navigation to the list = the proven pattern from `03/04/05`: `launchApp clearState` →
  `evalScript` unique email → `runFlow ../subflows/register-user.yaml` (auto-login) →
  `extendedWaitUntil ".*Job Offers.*"`. Flows 01/02 are **read-only on the list** — no `apply-to-offer`,
  no new subflow.
- Selectors: full-node-text DOTALL regex, tokens wrapped `.*…​.*` (per `CLAUDE.md` selector notes).
- `scrollUntilVisible` on a company anchor (same primitive as `apply-to-offer`).
- Mechanic-proof-then-red pattern from `04/05`: all GREEN proofs first, `takeScreenshot`, then the
  single RED spec assertion **last** (Maestro aborts on first failure).
- `config.yaml` scopes the workspace to `flows/` → 01/02 are auto-included (verify it does not
  enumerate flows explicitly; if it does, add 01/02).

## Files to create

### `flows/01-offers-locations.yaml` — R1 / BUG-A (C1)

```yaml
# R1 — offers list: multi-location offers must render a COMMA-SEPARATED list (spec). Buggy
# concatenates with no separator ("WarsawGdansk") -> adjudicated BUG-A (C1), list surface.
#
# Asserts the SPEC; a RED here is the CORRECT result = the finding. We do NOT assert the run-together
# string to force green (that rubber-stamps BUG-A — a disqualifying move). Before the red it PROVES
# (green) the list loaded its 5 offers and a clean tile renders correctly, then reaches the BigTech
# tile — so a red is the app violating spec, not the flow failing to drive it.
appId: com.example.buggy
---
- launchApp:
    clearState: true
- evalScript: ${output.email = 'qa.user.' + Date.now() + '@example.com'}
- runFlow:
    file: ../subflows/register-user.yaml
    env:
      EMAIL: "${output.email}"
      PASSWORD: "ValidPass123"
      CONFIRM: "ValidPass123"
- extendedWaitUntil:
    visible: ".*Job Offers.*"
    timeout: 8000
# --- Mechanic proof (GREEN): a CLEAN tile renders correctly (single loc + formatted range). ---
- assertVisible: ".*StartupXYZ.*"
- assertVisible: ".*Krakow.*"                 # single location renders as one clean value
- assertVisible: ".*Salary: 5000-8000.*"      # a normal range renders correctly
# --- Mechanic proof (GREEN): the list loaded all 5 offers (company anchors, top -> bottom). ---
- scrollUntilVisible: { element: { text: ".*TechCorp.*" }, direction: DOWN }
- scrollUntilVisible: { element: { text: ".*StartupXYZ.*" }, direction: DOWN }
- scrollUntilVisible: { element: { text: ".*BigTech.*" }, direction: DOWN }
- scrollUntilVisible: { element: { text: ".*MobileFirst.*" }, direction: DOWN }
- scrollUntilVisible: { element: { text: ".*QualityAssurance.*" }, direction: DOWN }
# --- Mechanic proof (GREEN): re-position to the BUG-A target tile (mid-list, scrolled past above). ---
- scrollUntilVisible: { element: { text: ".*BigTech.*" }, direction: UP }
- assertVisible: ".*BigTech.*"                # reached the target tile
- takeScreenshot: "docs/runs/screens/01-offers-locations"
# --- SPEC assertion (the FINDING): multi-location shown comma-separated. ---
# BigTech is Warsaw + Gdansk. A correct render is "Warsaw, Gdansk"; Buggy shows "WarsawGdansk"
# -> this assertVisible FAILS -> RED = C1 (BUG-A, list). Expected red; do not weaken to match the app.
- assertVisible: ".*Warsaw, Gdansk.*"
```

### `flows/02-offers-salary.yaml` — R1 / BUG-C (C3)

```yaml
# R1 — offers list: an object-shaped salary must render as a formatted range, like every other offer
# (spec). Buggy leaks the raw JSON "{"min": 10000, "max": 15000}" -> adjudicated BUG-C (C3), list surface.
#
# Asserts the SPEC; a RED here is the CORRECT result = the finding. We do NOT assert the raw JSON to
# force green (that rubber-stamps BUG-C — a disqualifying move). Before the red it PROVES (green) a
# clean range renders correctly and the JSON tile is reached — so a red is the app violating spec.
appId: com.example.buggy
---
- launchApp:
    clearState: true
- evalScript: ${output.email = 'qa.user.' + Date.now() + '@example.com'}
- runFlow:
    file: ../subflows/register-user.yaml
    env:
      EMAIL: "${output.email}"
      PASSWORD: "ValidPass123"
      CONFIRM: "ValidPass123"
- extendedWaitUntil:
    visible: ".*Job Offers.*"
    timeout: 8000
# --- Mechanic proof (GREEN): a normal salary renders as a formatted range. ---
- assertVisible: ".*StartupXYZ.*"
- assertVisible: ".*Salary: 5000-8000.*"
# --- Mechanic proof (GREEN): reach the object-salary tile (bottom of the list). ---
- scrollUntilVisible: { element: { text: ".*QualityAssurance.*" }, direction: DOWN }
- assertVisible: ".*QualityAssurance Ltd.*"   # reached the target tile
- takeScreenshot: "docs/runs/screens/02-offers-salary"
# --- SPEC assertion (the FINDING): salary shown as a formatted range. ---
# Underlying data is min 10000 / max 15000 -> a correct render is "10000-15000 PLN". Buggy leaks raw
# JSON -> the range text is absent -> this assertVisible FAILS -> RED = C3 (BUG-C, list). Expected red.
- assertVisible: ".*Salary: 10000-15000.*"
```

## Autoatak (self-attack)

1. **Masking inside a flow.** Maestro aborts on the first failed assertion → each flow carries
   **exactly one** RED, placed **last**. That is precisely why BUG-B-list cannot be folded into 01/02
   (it would need its own RED → its own flow). ✔
2. **Mechanic-proof genuinely GREEN before the RED.** Each flow proves (a) on the list (`Job Offers`),
   (b) a clean tile renders correctly, (c) the target tile is reached — all green — before the spec
   RED. So a red = app violates spec, not flow-failed-to-drive. Mirrors `04/05`. ✔
3. **False-red risk (red for the wrong reason).** The RED target must be VISIBLE when asserted, else
   `assertVisible` fails by not-visible (wrong reason). Mitigation: the company-anchor scroll
   (`BigTech` UP in 01; `QualityAssurance` DOWN in 02) puts the tile in-viewport immediately before
   the red; the red then fails only because the spec string is absent. **Build-time check:** confirm
   the location/salary line of the target is in the same viewport as its company after the scroll
   (adjacent in the layout, ~70px below — expected yes; verify on the run). ✔
4. **False-green risk (red accidentally passes).** Verified against the dumps: neither
   `Warsaw, Gdansk` nor `Salary: 10000-15000` (comma / hyphen-range forms) appears **anywhere** in the
   list hierarchy. The location picker (which *does* hold per-location nodes like "Location option:
   Warsaw") is a modal we **never open** in 01/02 → no contamination. ✔
5. **Scroll mechanics / flake.** 02 is monotonic DOWN. 01 has one `direction: UP` scroll-back — same
   robust `scrollUntilVisible` primitive; verify under the suite's 3×/0-flake standard. ✔
6. **Selectors.** Company anchors are unique substrings; never anchor on the colliding title
   "Senior QA". Salary asserted with the `Salary:` label prefix (matches the a11y `label\nvalue`
   shape) to target the salary element specifically. ✔

## Documented gap (accepted)

R1's salary-**absence** rule (BUG-B on the list, `TechCorp` → `null`) has **no executable list
assertion** under this plan. It is covered by: the RED in `04-history-no-salary` (same root defect,
history surface) + the SPEC-MATRIX anti-case row (R1, R3). This is a deliberate, lead-approved
consequence of the 2-flow decision. **Build action:** do **NOT** add a note to SPEC-MATRIX — the
"null on the list" anti-case row already documents it. The human-facing gap note goes to the
**README on P5**, not this session.

## Verification (at build, not in plan mode — state-changing)

**Hard STOP on any divergence — never self-resolve silently.** If reality contradicts the plan at
any step below, stop and reconcile with the lead.

1. **Re-confirm ground-truth.** Fresh `maestro hierarchy` of the offers list; diff per-tile strings
   vs the 2026-06-14 dumps (expect identical — offline static app). Divergence → STOP + reconcile
   (plan-bridge rule), do not adjust assertions to whatever the new dump shows.
2. **Run each flow alone.** `maestro test flows/01-offers-locations.yaml` → all GREEN through the
   screenshot, then RED at `.*Warsaw, Gdansk.*`. `…/02-offers-salary.yaml` → RED at
   `.*Salary: 10000-15000.*`. Inspect run output + screenshot to confirm the red is the spec
   violation, not a drive failure.
3. **Screenshot framing (01) — load-bearing.** The `01-offers-locations` screenshot **MUST** frame
   the `Locations: WarsawGdansk` node, **not just** the `BigTech` company line — it is the BUG-A
   adjudication artifact. The `scroll … direction: UP` may leave the location line at the viewport
   edge; confirm on the actual run that the location node is fully captured. If it is clipped, STOP
   and reconcile the scroll target (e.g. anchor the UP scroll closer to the location line) — this is
   a mechanics fix, never an assertion change.
4. **Full suite.** Expect **8 flows: 4 green** (`03, 06, 07, 08`) **+ 4 red-findings** (`01, 02, 04,
   05`). Confirm 01/02 reds do not destabilize the green flows.
5. **Flake.** Run 01/02 ×3 (suite standard 0 flake), **special attention to 01's scroll-back-up**
   (02 is monotonic DOWN).
6. **Report, do not rule.** After the runs: suite report (green/red) + the screenshot and hierarchy
   paths for 01 and 02. Do **not** issue BUG/PASS — the human adjudicates (R1 = BUG already recorded).

## Out of scope / closeout

- No new subflow; no `apply-to-offer`; flows 01/02 are read-only on the list.
- **SPEC-MATRIX: one edit only** — set R1's `Flow` cell (`_TBD — P4_`) to `01, 02` (parity with P3:
  R2 → 03, R3 → 04+05). Do **not** add an anti-case row or a gap note — the R1 anti-cases
  (concatenation, null-on-list, raw-JSON) and `verdict = BUG` are already present.
- Closeout (End-of-Session Cycle): suite green/red report, update `CLAUDE.md` Current Status
  (R1 → flows 01+02; phase P4 done) + Session Log, update `PROGRESS.md`, `/export`, **archive this
  plan to `docs/ai-history/`**, update the ai-history index, local commit (separate logical units;
  push = human's call).
