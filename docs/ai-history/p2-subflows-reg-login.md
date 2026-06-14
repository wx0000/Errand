# P2 — Subflows + registration/login flows

> Plan bridge for Phase P2. Plan mode ON → design here → (accept) → switch OFF → build from this
> file. Per CLAUDE.md the build verifies this plan against the live emulator before implementing;
> if reality contradicts the plan, stop and reconcile. Archive this file to `docs/ai-history/` at
> End-of-Session (unconditional).
>
> **Plan-bridge filename:** the conventional name (CLAUDE.md `<phase>-<topic>.md`) is
> `p2-subflows-reg-login.md`. The rename cannot run in plan mode (non-read-only fs op) → it is
> **build step 0**: `mv` this file to `.claude/plans/p2-subflows-reg-login.md` before any flow is written.

## Context

P1 is closed: recon (`7a15bd4`) + adjudication (`cd4bf26`). R1–R5 are adjudicated — **R1 BUG,
R2 PASS, R3 BUG, R4 PASS, R5 PASS**. `recon.yaml` is proof-of-method, **not** part of the suite;
only its *conventions* carry forward.

P2 builds the **first real suite flows**: reusable subflows + the two PASS requirements that are
self-contained (no offers/apply/history dependency): **R4 (registration)** and **R5 (login/
logout)**. R4 and R5 are **PASS** per adjudication, so these three flows are **expected green**.
That does not change the cardinal rule: each flow asserts the **SPEC**. If one goes red, it is a
**finding for the human to adjudicate** — we never bend the assertion to match the app.

Scope is exactly: 4 subflows + 3 flows. Apply-to-offer, offers-list, and history flows are
**P3/P4** and out of scope here. The persistence anti-case and the anti-case table are filled with
the **bug** flows (R1/R3) in P3/P4 — **not now**.

## Directory layout (new)

```
config.yaml                              # Maestro workspace config — scopes the run to flows/
flows/
  06-registration-validation.yaml        # R4 — three validation errors, each ABOVE its field
  07-registration-autologin.yaml         # R4 — valid register auto-logs in → Job Offers
  08-login-logout.yaml                   # R5 — logout → login screen; re-login → Job Offers
subflows/
  register-user.yaml                     # attempt registration (params); NO outcome assertion
  login.yaml                             # submit login creds (params); NO outcome assertion
  logout.yaml                            # tap Logout; NO outcome assertion
  open-profile.yaml                      # open profile from header; NO history-content assertion
```

Flow numbering starts at **06**: 01–05 are reserved for the P3/P4 offers/apply/history flows.
Subflows live under `subflows/` and are **fragments** (invoked via `runFlow: file:`); they are
never run standalone — `config.yaml` (`flows: ["flows/*.yaml"]`) keeps them out of the suite run.

## Confirmed selector map (from real recon dumps — evidence, not guesses)

All strings below are quoted/derived from the hierarchy dumps named. Convention (P1): selectors
are **regex anchored to the FULL node text, DOTALL**; node text is `"label\nvalue"`, so wrap tokens
in `.*…*`; tap the **clickable `Button`**, never the wrapper `View`.

| Screen | Element | Selector | Source dump |
|---|---|---|---|
| Login | screen title | `.*Welcome Back.*` | `00-login.txt` n12 |
| Login | email field | `.*Email input field.*` | `00-login.txt` n14 (EditText) |
| Login | password field | `.*Password input field.*` | `00-login.txt` n15 (EditText) |
| Login | submit button | `Login` (exact → clickable Button n17, not wrapper "Login button" n16) | `00-login.txt` |
| Login | go-to-register | `.*Don't have an account.*` | `00-login.txt` n19 (Button) |
| Register | form heading | `.*Create Account.*` | `06-…txt` n46 |
| Register | email field | `.*Email input field.*` | `06-…txt` n48 (EditText) |
| Register | password field | `.*Password input field.*` | `06-…txt` n49 (EditText) |
| Register | confirm field | `.*Confirm password input field.*` | `06-…txt` n50 (EditText) |
| Register | submit button | `Register` (exact → clickable Button n52, not wrapper "Register button" n51) | `06-…txt` |
| Register | **email error** | `.*Please enter a valid email address.*` | `06-reg-email-error.txt` n47, top-y **936** |
| Register | **password error** | `.*Password must be at least 8 characters.*` | `07-reg-pwd-error.txt` n48, top-y **1135** |
| Register | **confirm error** | `.*Passwords do not match.*` | `08-reg-confirm-error.txt` n49, top-y **1335** |
| Profile | screen title | `.*Profile screen title.*` | `09-history.txt` n45 |
| Profile | open from header | `.*Profile button.*` | recon login-logout act (worked) |
| Profile | logout button | `Logout` (exact → clickable Button n70, not wrapper "Logout button" n69) | `09-history.txt` |

**Position evidence for R4 (error ABOVE field):** error top-y < field top-y in every dump —
email 936 < input 1002; password 1135 < input 1201; confirm 1335 < input 1401. The flow asserts
this with Maestro's **relative `above:` selector** (below), the programmatic equivalent of the
recon y-comparison.

## Subflows — boundary rule

> **Each subflow asserts ONLY its own name (a readiness gate for its own action) — nothing beyond.
> Requirement-level outcomes are asserted in the FLOW, never inside a subflow.** This is exactly
> why `register-user` does not assert success: the *flow* decides whether success (07) or failure
> (06) is expected.

### `subflows/register-user.yaml`  — params: `EMAIL`, `PASSWORD`, `CONFIRM`  (Decision A)
- `env:` defaults block declaring the params (empty defaults). **Caller MUST pass all three** —
  an empty `CONFIRM` would mismatch the password and fail registration, producing a **false red**
  on R4/R5. Every call site below passes `CONFIRM` explicitly.
- Steps: tap `.*Don't have an account.*` → **gate** `assertVisible: .*Create Account.*` (own scope:
  reached the register form; auto-waits, no sleep) → fill email/password/confirm → tap `Register`.
- **Asserts nothing about the outcome** (no "Job Offers", no error). Caller asserts.

### `subflows/login.yaml`  — params: `EMAIL`, `PASSWORD`
- `env:` defaults block.
- Steps: **gate** `assertVisible: .*Welcome Back.*` (own scope: on login screen) → fill email/
  password → tap `Login`. **No outcome assertion.**

### `subflows/logout.yaml`  — no params
- Steps: **gate** `assertVisible: Logout` (own scope: logout control present) → tap `Logout`.
  **No outcome assertion** (landing on login is the flow's R5 assertion).

### `subflows/open-profile.yaml`  — no params
- Steps: tap `.*Profile button.*` → **gate** `assertVisible: .*Profile screen title.*` (own scope:
  profile opened). **Asserts no history content** — entry-content assertions are R3 (P3).

## Flows — assertions tied to the SPEC

Every flow: starts with `launchApp: { clearState: true }`; `appId` in the flow config header
(value `com.example.buggy`); **no `sleep`, no hardcoded wait** — `assertVisible` auto-waits, and
every `extendedWaitUntil` carries a one-line comment justifying it. **No `tags:`** — tags are on
this project's explicitly-rejected extras list (CI/JUnit, tags, iOS/Appium, perf, visual); they go
to the README as future work, not into suite code. Selective-run is out of P2 scope.

### `flows/06-registration-validation.yaml`  — R4 (validation)
Three isolated sub-scenarios; each **relaunches clean** (`launchApp clearState`) for isolation, then
`runFlow register-user` with the three field values (Decision A: `register-user` is the generic
registration-attempt helper → reused by **all three** cases), then asserts. Each case makes exactly
**one** field invalid (the other two valid). Each is backed by its own recon dump (06/07/08), which
show **exactly one** error node — so we also assert the other two errors are **absent** (spec: a
valid field raises no error).

1. **Invalid email** — email `notanemail`, password `ValidPass123`, confirm `ValidPass123`:
   ```yaml
   - assertVisible:                                   # spec: error ABOVE the email field
       text: ".*Please enter a valid email address.*"
       above: { text: ".*Email input field.*" }
   - assertNotVisible: ".*Password must be at least 8 characters.*"
   - assertNotVisible: ".*Passwords do not match.*"
   ```
2. **Short password** — email valid, password `short` (5), confirm `short`: assert
   `.*Password must be at least 8 characters.*` **above** `.*Password input field.*`; assert the
   email + confirm errors absent.
3. **Confirm mismatch** — email valid, password `ValidPass123`, confirm `Different456`: assert
   `.*Passwords do not match.*` **above** `.*Confirm password input field.*`; assert the email +
   password errors absent.

No account is ever created (validation blocks submit) → email uniqueness is irrelevant here; uses
a valid-format constant where a valid email is needed.

### `flows/07-registration-autologin.yaml`  — R4 (auto-login)
- `evalScript` → unique email `qa.user.<timestamp>@example.com` (hard rule: unique per run).
- `runFlow register-user` (EMAIL=unique, PASSWORD=`ValidPass123`, **CONFIRM=`ValidPass123`**) —
  no assertion inside. CONFIRM passed explicitly (= PASSWORD) so registration succeeds.
- **R4 assertion:** `extendedWaitUntil { visible: ".*Job Offers.*", timeout: 8000 }`
  *(comment: registration auto-logs in and navigates to the authenticated home; allow for the
  transition — 8 s as observed in recon).* Landing on Job Offers **with no login step** is the
  observable proof of auto-login.

### `flows/08-login-logout.yaml`  — R5
- `evalScript` → unique email (captured **once**, reused for register **and** re-login).
- `runFlow register-user` (EMAIL, PASSWORD=`ValidPass123`, **CONFIRM=`ValidPass123`**) — no
  assertion inside. CONFIRM passed explicitly (= PASSWORD) so registration succeeds.
- `extendedWaitUntil { visible: ".*Job Offers.*" }` *(comment: readiness only — confirm we are
  registered + auto-logged-in before exercising logout; this is setup, not the R5 assertion).*
- `runFlow open-profile` → `runFlow logout`.
- **R5 assertion 1:** `extendedWaitUntil { visible: ".*Welcome Back.*" }` *(comment: spec — logout
  returns to the login screen)*.
- `runFlow login` (same EMAIL, PASSWORD).
- **R5 assertion 2:** `extendedWaitUntil { visible: ".*Job Offers.*" }` *(comment: spec — re-login
  with the registration credentials reaches the offers list)*.

This mirrors the recon `login-logout` act, which passed (R5 = PASS) — the flow re-expresses it as
suite subflows + spec assertions.

## Hard rules — checklist (from the task)

- [x] `clearState` at the start of **every** flow (06 also relaunches clean before each of its 3 cases).
- [x] Unique email per run via timestamp (`evalScript`) — in 07 and 08 (the flows that create an account).
- [x] **Zero** `sleep` / hardcoded waits; `assertVisible` auto-waits; each `extendedWaitUntil` is commented.
- [x] One flow = one requirement (06→R4 validation, 07→R4 auto-login, 08→R5).
- [x] `appId: com.example.buggy` in each flow's config header; `config.yaml` scopes the suite to `flows/`.
- [x] Selectors reuse recon conventions; **`recon.yaml` does not enter the suite** — only its conventions.
- [x] Subflows assert only their own name; requirement outcomes asserted in the flow.

## Resolved decision — `register-user` signature = **A**

`register-user(EMAIL, PASSWORD, CONFIRM)` — generic "attempt registration with these values."
Flow 06 reuses it for **all three** validation cases; flows 07/08 pass `CONFIRM=PASSWORD`. Most DRY;
fully realizes the "no outcome assertion inside" boundary intent (the failure flow reuses the same
helper and asserts the failure itself). Adds `CONFIRM` beyond the literal "email+password" wording.
_(Human deferred the selection; A was the recommended option — correctable at the approval gate.)_

Rejected: **B** `EMAIL, PASSWORD` (confirm=password internally) — would force the confirm-mismatch
case to inline fill (asymmetric flow 06). **C** happy-path-only helper + fully inline flow 06 —
duplicates navigation/fill across the three cases.

## Build-time verification (before/while implementing — not blind execution)

1. **`above:` relative selector** behaves on the emulator for the three R4 errors (primary position
   check). If it proves flaky, fall back to presence assertion + a screenshot as position evidence,
   but position is core to R4 — try `above:` first.
2. **`evalScript` + `Date.now()`** is available in Maestro's JS runtime for the unique email.
3. **`config.yaml`** correctly scopes `maestro test .` to `flows/` only (subflows excluded); confirm
   whether a config-level default `appId` is supported — if not, keep `appId` in each flow header.
4. Run the three flows green end-to-end; any red is a **finding** (adjudicate), not an assertion to bend.

## Deferred (NOT this phase)

- `subflows/apply-to-offer.yaml` and flows 01–05 (offers list, apply, history) → **P3/P4**.
- Persistence anti-case + the anti-case table → filled with the **bug** flows (R1/R3) in P3/P4.
- Linking red-run Maestro artifacts into `BUGS.md` → **P5**.
- Rejected extras (CI/JUnit, tags + selective-run, iOS/Appium, perf, visual) → **README future
  work**, never suite code.
