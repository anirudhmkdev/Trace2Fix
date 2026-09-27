# Prompt 1

Update the existing `PROTOTYPE_PLAN.md` for Trace2Fix with the following corrections.

This task is **PLAN REVISION ONLY**.

Do NOT implement the application, create source files, install dependencies, or begin coding.

Keep the existing prototype scope, repository structure, technology stack, and implementation order unless a change below requires a small adjustment.

## Required Changes

### 1. Make the seeded production failure less revealing

The current plan says that missing `EXCHANGE_RATE` causes:

`config.get("EXCHANGE_RATE")` → `KeyError`

Change this design.

The production configuration should still intentionally omit `EXCHANGE_RATE`, but configuration access should not directly produce an obvious `KeyError: EXCHANGE_RATE`.

Instead, the missing value should propagate into the payment/currency-conversion path and cause a less explicit runtime failure.

Recommended behavior:

* `config.get("EXCHANGE_RATE")` returns `None` when the key is missing.
* Payment/currency conversion assumes the rate exists.
* Currency calculation attempts something equivalent to:
`amount * exchange_rate`
* Because `exchange_rate` is `None`, runtime execution fails with a `TypeError`.
* The API consequently returns HTTP 500.

The production log may reveal a `float`/`NoneType` conversion-related failure, but it must NOT directly state:

* `EXCHANGE_RATE is missing`
* `missing configuration caused the error`
* the final root cause.

The goal is to require evidence correlation.

The evidence should be distributed like this:

**Log evidence**

* payment request failed,
* currency-conversion stage was involved,
* runtime `TypeError`,
* request ID and execution sequence.

**Code evidence**

* currency conversion reads a rate from configuration,
* application assumes that rate is present,
* missing-value validation does not exist.

**Configuration/documentation evidence**

* `prod.yaml` lacks the required value,
* documentation establishes what configuration currency conversion expects.

**Test evidence**

* correctly configured payment scenarios are covered,
* the missing-configuration scenario is not covered.

The complete root cause should therefore require correlation across multiple investigation agents.

---

### 2. INC-001 must not reveal or strongly suggest the root cause

Update the plan so:

`incidents/INC-001.md`

contains only information an on-call developer would realistically have at the start of an incident.

It may include:

* incident ID,
* timeline,
* affected endpoint,
* HTTP status,
* request ID,
* user/customer impact,
* observed symptoms,
* relevant log location.

It must NOT mention:

* `EXCHANGE_RATE`,
* missing production configuration,
* currency configuration being the suspected cause,
* the true affected configuration key,
* any strong root-cause hypothesis.

A vague description such as:

`Payment processing failure requiring investigation`

is acceptable.

---

### 3. README must not spoil the investigation

Update the README requirement.

`README.md` should contain:

* project setup,
* dependency installation,
* normal test commands,
* development server commands,
* how to reproduce INC-001,
* basic Trace2Fix usage.

It must NOT explain the root cause of INC-001.

It can say that the repository contains an intentionally seeded production incident, but must not say what the defect is.

---

### 4. Add deterministic incident reproduction

Add:

`scripts/reproduce_incident.py`

to the repository plan.

Its purpose is to reproduce INC-001 deterministically without requiring the user to manually start Uvicorn and send a separate curl request.

The script should:

1. run the application using production configuration,
2. issue the affected payment request,
3. reproduce the same production-style failure,
4. print useful output showing that the request resulted in HTTP 500.

Prefer FastAPI `TestClient` or another simple local mechanism.

The script itself must not print or reveal the root cause.

README may still include manual Uvicorn/curl commands as optional instructions.

---

### 5. Keep configuration testable

Update the implementation notes so environment/configuration loading can be switched reliably between `dev` and `prod`.

Avoid architecture where `APP_ENV` is permanently captured at module-import time in a way that makes tests or `reproduce_incident.py` unreliable because of Python module caching.

Keep the implementation simple.

---

### 6. Change the documentation remediation requirement

The current plan says Trace2Fix will automatically:

`update docs/configuration.md`

after fixing INC-001.

Change this.

Trace2Fix should instead:

1. inspect documentation impact,
2. determine whether documentation needs modification,
3. update documentation only if the confirmed remediation changes or clarifies documented behavior.

Documentation changes must not be artificially forced just to satisfy the workflow.

The final incident report should record:

`Documentation impact: update required / no update required`

with a short reason.

---

## Update the Primary Incident Behavior section

Revise it approximately around this model:

* **Bug area:** interaction between production configuration and payment/currency-conversion logic.
* **Trigger:** currency-conversion payment request while `APP_ENV=prod`.
* **Production condition:** required rate value is unavailable.
* **Runtime symptom:** currency calculation receives a missing value and fails with a `TypeError`.
* **External symptom:** HTTP 500.
* **Logs:** expose runtime symptoms and execution context but not the complete root cause.
* **Code:** exposes the unsafe configuration assumption.
* **Configuration/docs:** reveal the missing required value and expected behavior.
* **Tests:** reveal that this edge case is not covered.

Do not describe the incident as a simple `KeyError`.

---

## Update Acceptance Criteria

The revised plan must include these criteria:

* `pytest` under normal/dev configuration passes.
* Development currency payment succeeds.
* Production reproduction script deterministically results in HTTP 500.
* The runtime failure is less explicit than `KeyError: EXCHANGE_RATE`.
* `INC-001.md` does not reveal or strongly suggest the root cause.
* `README.md` does not reveal the root cause.
* The logs contain meaningful evidence but do not directly state the root cause.
* The codebase, configuration, documentation, and tests each contribute distinct evidence useful to the Trace2Fix investigation.
* `scripts/reproduce_incident.py` exists in the planned repository structure.
* No database, Docker, frontend, microservices, or unnecessary infrastructure is added.

---

## Important Constraints

Preserve the project's small hackathon scope.

Do NOT add:

* databases,
* Docker,
* frontend,
* authentication,
* external APIs,
* OpenTelemetry,
* GitHub integration,
* additional incidents,
* additional services,
* unnecessary abstractions.

Do not implement anything yet.

Only update `PROTOTYPE_PLAN.md`.

After updating the file, briefly summarize the sections you changed and then stop.

---

# Prompt 2 (Agent-mode)

Implement the prototype foundation described in `PROTOTYPE_PLAN.md`.

This task should build the complete sample application and incident environment.

Do NOT implement the Trace2Fix Bob skill yet.

Do NOT fix INC-001.

## Important demo-integrity rules

`PROTOTYPE_PLAN.md` contains implementation knowledge about the seeded incident because it is being used during construction.

However, the actual runtime project artifacts must NOT directly reveal the root cause.

Therefore:

1. `config/prod.yaml` must simply omit the relevant configuration value.
* Do NOT add comments explaining that a value is intentionally missing.
* Do NOT write comments such as `seeded bug`, `intentional defect`, or similar.


2. Application source code must look like normal application code.
* Do NOT add comments identifying intentionally unsafe code.
* Do NOT mention INC-001 inside application source.
* Do NOT explain the hidden defect in comments or docstrings.


3. `incidents/INC-001.md` must contain only:
* incident ID
* timeline
* affected endpoint
* HTTP status
* request ID
* user/customer impact
* observed symptoms
* location of relevant logs


It must NOT mention:
* `EXCHANGE_RATE`
* missing configuration
* configuration as a suspected cause
* the affected source line
* the final root cause


4. `incidents/INC-001-logs.json` must provide useful runtime evidence without explaining the cause.
5. `README.md` may state that the repository contains a seeded production incident for Trace2Fix demonstration purposes, but it must NOT reveal what causes INC-001.
6. `AGENTS.md`, if created during this task, may state:
`Do not repair seeded incidents unless explicitly requested during remediation.`
It must NOT describe the root cause of INC-001.

These restrictions are important because the future Trace2Fix agents must discover the problem through evidence correlation rather than reading an answer embedded in the repository.

---

## Build Requirements

Implement the repository described in `PROTOTYPE_PLAN.md`.

Keep the application intentionally small.

### 1. Dependencies

Create `requirements.txt` with only the dependencies needed for:

* FastAPI
* Uvicorn
* PyYAML
* Pydantic
* Pytest
* HTTPX / FastAPI TestClient support

Do not add unnecessary packages.

---

### 2. Application

Implement:

`app/__init__.py`

`app/main.py`

`app/config.py`

`app/payment.py`

`app/models.py`

Required API behavior:

#### Health endpoint

Add a simple health endpoint such as:

`GET /health`

Expected response:

HTTP 200.

#### Order endpoint

Implement the minimal order endpoint described in the plan.

Do not add persistence or a database.

#### Payment endpoint

Implement:

`POST /payments`

It should accept approximately:

* order ID
* amount
* currency

For normal development configuration:

* USD/basic payment behavior should work.
* currency-conversion payment should work.
* response should return HTTP 200.

---

### 3. Configuration

Create:

`config/dev.yaml`

`config/prod.yaml`

Development configuration should include all required values.

Production configuration should look like an ordinary production configuration file.

Do NOT put comments in `prod.yaml` explaining what is missing.

Configuration loading must support reliably selecting `dev` or `prod`.

Do not permanently capture `APP_ENV` at module import in a way that makes tests or the reproduction script unreliable.

Prefer a small function or lightweight configuration object whose environment can be selected when needed.

When an optional lookup does not exist, `config.get()` should be capable of returning `None`.

Do not add validation that repairs INC-001 yet.

---

### 4. Payment / Currency Conversion

Implement the payment/currency conversion path described in the plan.

Normal configured currency conversion must succeed.

When production configuration lacks the value needed by currency conversion, the missing value should eventually reach the arithmetic operation and cause a runtime `TypeError`.

The exception should be similar in nature to:

`float * None`

The runtime error must NOT be:

`KeyError: EXCHANGE_RATE`

Do not intentionally catch and fix this condition yet.

This is the seeded production defect required for the future Trace2Fix investigation.

---

### 5. Existing Tests

Create:

`tests/conftest.py`

`tests/test_orders.py`

`tests/test_payments.py`

Tests should represent the application's existing pre-incident test suite.

They should cover normal development-config behavior.

They should NOT contain a regression test for missing production configuration.

Running:

`pytest`

must pass completely.

Do not create the INC-001 regression test yet.

That test will be created later by Trace2Fix during remediation.

---

### 6. Incident

Create:

`incidents/INC-001.md`

Make it realistic but concise.

Use a request ID such as:

`req-1842`

Describe something equivalent to:

* customer payment request failed
* POST /payments
* HTTP 500
* observed production time window
* payment processing affected
* investigation required

Do not reveal the cause.

---

### 7. Incident Logs

Create:

`incidents/INC-001-logs.json`

Requirements:

* at least 40 structured log entries
* realistic noise from normal application activity
* multiple request IDs
* the relevant request ID appears several times
* payment processing begins
* currency-conversion stage appears
* runtime TypeError appears
* HTTP 500 appears
* relevant evidence is not the first log entry
* logs do not say that configuration is missing
* logs do not name the root cause

The log investigator should need to filter/correlate events.

---

### 8. Documentation

Create:

`docs/architecture.md`

`docs/payment-flow.md`

`docs/api.md`

`docs/configuration.md`

Keep each document short.

Documentation must reference real project behavior.

The documents should collectively explain:

* application structure
* payment execution path
* API contract
* expected configuration

`docs/configuration.md` should document required configuration normally, including the configuration expected by currency conversion.

Do NOT say that production is missing it.

The documentation should describe intended behavior, not explain INC-001.

---

### 9. Deterministic Reproduction Script

Create:

`scripts/reproduce_incident.py`

The script must reproduce INC-001 without requiring a manually started Uvicorn server.

Prefer FastAPI `TestClient`.

Required behavior:

1. select production configuration reliably
2. invoke the affected payment endpoint
3. use a non-base currency requiring conversion
4. observe the production failure
5. print:
* request being executed
* HTTP status
* response



Expected result:

HTTP 500.

The script must not print:

* missing configuration
* `EXCHANGE_RATE is missing`
* root-cause explanations

Its purpose is reproduction, not diagnosis.

Ensure expected server exception behavior is handled so the script can complete and clearly display the simulated HTTP 500 rather than simply terminating with an uncaught Python traceback.

---

### 10. Benchmark Foundation

Create:

`benchmark/benchmark.py`

`benchmark/results.md`

For this phase, implement the simple benchmark functionality described in `PROTOTYPE_PLAN.md`.

At minimum:

* start timer
* stop timer
* investigator/workflow name
* incident ID
* elapsed seconds

Keep it simple.

More detailed Trace2Fix metrics will be added later.

---

### 11. README

Create `README.md`.

Include:

* project purpose at a high level
* installation
* normal test command
* development server command
* production server command
* reproduction script command
* optional curl example
* statement that INC-001 is a seeded production incident used for the Trace2Fix workflow

Do NOT disclose the root cause of INC-001.

A reader should be able to reproduce the failure without knowing why it occurs.

---

## Verification

After implementation, perform these checks.

### Check 1

Run:

`pytest`

Expected:

all existing tests pass.

### Check 2

Verify a normal development currency payment succeeds.

Expected:

HTTP 200.

### Check 3

Run:

`python scripts/reproduce_incident.py`

Expected:

HTTP 500.

### Check 4

Confirm the production runtime failure is caused by a `TypeError` rather than a revealing `KeyError`.

### Check 5

Inspect:

* `INC-001.md`
* `README.md`
* `INC-001-logs.json`
* actual source comments
* actual configuration comments

Ensure none directly disclose the root cause.

### Check 6

Verify each evidence domain contributes something different:

Logs:
runtime symptom and execution sequence.

Code:
unsafe assumption around configuration value.

Configuration/docs:
actual vs expected configuration.

Tests:
missing edge-case coverage.

---

## Constraints

Do NOT:

* implement Trace2Fix skill yet
* fix INC-001
* create the missing regression test
* create additional incidents
* add a database
* add Docker
* build frontend/UI
* create CI/CD
* add OpenTelemetry
* add external APIs
* create microservices
* add authentication
* perform unrelated refactoring
* use subagents for this implementation task

Use the smallest reasonable implementation.

If you encounter an unintended implementation issue that prevents the acceptance criteria from passing, repair that issue.

Do not repair the intentionally seeded INC-001 defect.

At the end, report only:

1. files created,
2. test result,
3. development payment result,
4. incident reproduction result,
5. confirmation that INC-001 remains intentionally unresolved.

Then stop.

---

# Prompt 3 :

Implement the Trace2Fix workflow infrastructure for the existing repository.

This is **Prompt 3 / Agent Mode**.

Do NOT investigate INC-001 yet.

Do NOT fix INC-001.

Do NOT modify the intentional seeded defect.

Do NOT create the INC-001 regression test yet.

The goal of this task is only to create:

1. the reusable IBM Bob Trace2Fix skill,
2. the incident report template/supporting files if needed,
3. enhanced benchmark and metrics tooling.

---

# 1. Create the Trace2Fix Bob Skill

Create:

```text
.bob/skills/trace2fix/SKILL.md
```

Use valid Bob skill front matter.

Use approximately:

```yaml
---
name: trace2fix
description: Investigate production incidents using independent log, code, documentation/configuration, and test analysis, then perform evidence-backed remediation after explicit human approval.
user-invocable: true
---
```

Keep the skill focused and optimized for reuse.

Do not embed the known root cause of INC-001 anywhere in the skill.

The skill must work from the supplied incident evidence rather than from prior knowledge of this seeded incident.

---

# 2. Trace2Fix Investigation Phase

When the skill is invoked for an incident, the parent Bob agent should first read:

- the requested incident file,
- repository guidance in `AGENTS.md`,
- only the minimum initial context necessary to dispatch investigators.

The parent must NOT modify application source, tests, configuration, or documentation during investigation.

Then create exactly four independent investigation roles.

## Investigator A — Log Investigator

Purpose:

Analyze runtime evidence only.

Primary responsibilities:

- inspect the relevant incident log,
- identify request IDs,
- identify error events,
- reconstruct the event sequence,
- identify runtime symptoms,
- identify components mentioned by the logs.

Return only a concise structured result:

```text
AGENT: Log Investigator

OBSERVATIONS
- ...

EVIDENCE
- file/reference
- ...

HYPOTHESES
- ...

UNKNOWNS
- ...
```

The log investigator must not inspect application source, tests, or documentation unless absolutely required to interpret basic file formats.

---

## Investigator B — Code Investigator

Purpose:

Trace the observed runtime behavior through source code.

Responsibilities:

- identify the affected endpoint,
- follow the relevant execution path,
- inspect related functions,
- identify unsafe assumptions or failure points,
- identify affected files.

Return:

```text
AGENT: Code Investigator

EXECUTION PATH
- ...

OBSERVATIONS
- ...

EVIDENCE
- file/reference
- ...

HYPOTHESES
- ...

UNKNOWNS
- ...
```

The code investigator should focus on application source code and should not rely on conclusions from the other agents.

---

## Investigator C — Documentation + Configuration Investigator

Purpose:

Determine intended behavior and configuration expectations.

Responsibilities:

- inspect relevant documentation,
- inspect relevant environment configuration,
- compare documented expectations with available configuration,
- identify mismatches,
- identify whether documentation itself appears outdated.

Return:

```text
AGENT: Documentation + Configuration Investigator

EXPECTED BEHAVIOR
- ...

OBSERVATIONS
- ...

EVIDENCE
- file/reference
- ...

MISMATCHES
- ...

HYPOTHESES
- ...

UNKNOWNS
- ...
```

Do not modify documentation or configuration during investigation.

---

## Investigator D — Test Investigator

Purpose:

Assess existing test coverage.

Responsibilities:

- inspect relevant automated tests,
- identify scenarios currently covered,
- identify likely missing edge cases,
- determine whether the reported incident condition appears to have regression coverage,
- recommend a regression-test scenario without creating the test yet.

Return:

```text
AGENT: Test Investigator

EXISTING COVERAGE
- ...

MISSING COVERAGE
- ...

EVIDENCE
- file/reference
- ...

RECOMMENDED REGRESSION TEST
- ...

UNKNOWNS
- ...
```

Do not create or modify tests during investigation.

---

# 3. Independent / Parallel Investigation

Where supported by Bob, run the four investigators in parallel.

Use read-only/exploration-oriented subagents where possible.

Important:

- investigators should work independently,
- one investigator should not receive another investigator's conclusions before finishing,
- investigators should return concise summaries rather than large narrative reports,
- the parent agent should perform the final correlation.

Do not spawn extra subagents unless one of the four investigators genuinely requires a narrowly scoped supporting lookup.

Prefer exactly four investigation subagents.

---

# 4. Evidence Synthesis

After the four investigators return, the parent agent must correlate their findings.

The parent should identify:

- where agents agree,
- where evidence conflicts,
- which facts are directly observed,
- which statements are still hypotheses,
- whether there is enough evidence to identify a root cause.

The parent must not treat speculation as confirmation.

Important root-cause claims should reference repository evidence.

---

# 5. Root Cause Report

Before any remediation, produce a structured:

```text
TRACE2FIX ROOT CAUSE REPORT
```

Include:

```text
Incident

Observed Symptoms

Runtime Sequence

Relevant Execution Path

Observed Facts

Root Cause

Supporting Evidence

Affected Components

Missing Test Coverage

Recommended Regression Test

Proposed Remediation

Confidence

Remaining Unknowns
```

Confidence may be:

```text
HIGH
MEDIUM
LOW
```

Use:

- HIGH only when multiple evidence sources support the conclusion,
- MEDIUM when evidence strongly suggests the cause but something remains unverified,
- LOW when the conclusion is still mostly hypothetical.

---

# 6. Mandatory Human Approval Gate

After the root-cause report:

STOP.

Do not edit files.

Do not create tests.

Do not fix code.

Output clearly:

```text
STATUS: AWAITING REMEDIATION APPROVAL
```

Require an explicit developer instruction such as:

```text
Root cause approved. Proceed with remediation.
```

before moving forward.

---

# 7. Remediation Phase

Only after explicit human approval:

## Step 1 — Regression Test

Create the smallest regression test that reproduces the confirmed defect.

Where practical:

1. create the test,
2. run the test before changing application code,
3. confirm and record the expected failure.

Do not create unrelated tests.

---

## Step 2 — Minimal Fix

Apply the smallest reasonable remediation supported by the confirmed evidence.

Rules:

- avoid unrelated refactoring,
- do not redesign the application,
- preserve unaffected behavior,
- change configuration only if the approved remediation requires it,
- change documentation only if the remediation changes or clarifies documented behavior.

---

## Step 3 — Verification

Run:

1. new regression test,
2. relevant subsystem tests,
3. complete Pytest suite.

Then inspect the final diff.

Check:

- documentation impact,
- configuration impact,
- unrelated modifications.

---

# 8. Verification Completeness

Trace2Fix should evaluate these ten checks:

```text
1. Root cause supported by evidence
2. Defect reproduced
3. Regression test created
4. Regression test passes after remediation
5. Relevant subsystem tests pass
6. Full test suite passes
7. Documentation impact checked
8. Configuration impact checked
9. Changed files reviewed
10. No unrelated modifications detected
```

Calculate:

```text
completed applicable checks / total applicable checks × 100
```

Do not hardcode 100%.

A failed verification step must remain failed.

If a required verification check fails, do not mark the incident VERIFIED.

---

# 9. Final Incident Report

Create a reports directory if it does not already exist:

```text
trace2fix/reports/
```

After remediation, generate:

```text
trace2fix/reports/<INCIDENT-ID>.md
```

Required sections:

```text
# Trace2Fix Incident Report

## Incident

## Observed Symptoms

## Investigation Summary

## Root Cause

## Supporting Evidence

## Affected Components

## Regression Test

## Before-Fix Test Result

## Remediation

## Files Changed

## Targeted Test Results

## Full Test Suite Result

## Documentation Impact

## Configuration Impact

## Verification Checklist

## Verification Completeness

## Remaining Risks
```

Do not generate an INC-001 final report during this implementation task.

Only create supporting templates if useful.

---

# 10. Enhance Benchmark Tooling

The repository currently contains:

```text
benchmark/benchmark.py
benchmark/results.md
```

Extend the benchmark tooling instead of creating a competing measurement system.

Keep it lightweight.

The benchmark tool must support recording:

```text
incident_id
workflow
investigation_time_seconds
resolution_time_seconds
manual_steps
root_cause_correct
fix_attempts
regression_test_created
verification_completed
verification_total
verification_percentage
```

Supported workflows:

```text
manual
trace2fix
```

Persist structured results to:

```text
benchmark/results.csv
```

If the CSV does not exist, create it with headers automatically.

Prefer Python standard library modules such as:

- csv,
- argparse,
- datetime/time,
- pathlib.

Do not add a database or extra dependency.

---

# 11. Benchmark CLI

Keep the CLI simple.

Support a usable workflow such as:

```bash
python benchmark/benchmark.py start \
  --workflow manual \
  --incident INC-001 \
  --phase investigation
```

and:

```bash
python benchmark/benchmark.py stop \
  --workflow manual \
  --incident INC-001 \
  --phase investigation
```

Also provide a straightforward way to record the final metrics.

For example:

```bash
python benchmark/benchmark.py record \
  --workflow manual \
  --incident INC-001 \
  --manual-steps 15 \
  --root-cause-correct yes \
  --fix-attempts 2 \
  --regression-test-created yes \
  --verification-completed 6 \
  --verification-total 10
```

You may slightly simplify the CLI if another design is cleaner.

The important requirement is that it can record both timing and final benchmark values into `benchmark/results.csv`.

---

# 12. Benchmark Calculations

Automatically calculate:

```text
verification_percentage
```

from:

```text
verification_completed / verification_total
```

If both manual and Trace2Fix results exist for the same incident, it is acceptable to provide a small summary command such as:

```bash
python benchmark/benchmark.py compare --incident INC-001
```

which prints:

- investigation time difference,
- resolution time difference,
- manual-step difference,
- fix attempts,
- verification completeness.

Keep this optional if it adds unnecessary complexity.

Do not build charts or a dashboard.

---

# 13. README Update

Add only a short section describing:

- how to invoke/use the Trace2Fix skill,
- investigation → approval → remediation flow,
- benchmark commands.

Do not reveal the root cause of INC-001.

Do not add verbose documentation.

---

# 14. Demo Integrity

Do not read or use `PROTOTYPE_PLAN.md` as incident evidence when designing Trace2Fix behavior.

The skill must be generic.

Do not hardcode:

- INC-001 root cause,
- specific affected source lines,
- specific missing configuration values,
- expected answer.

Trace2Fix should discover incident causes dynamically from repository evidence.

Do not modify:

```text
app/
config/
tests/
incidents/INC-001.md
incidents/INC-001-logs.json
```

unless needed solely to repair an unintended blocker in the skill infrastructure.

In particular:

DO NOT FIX INC-001.

---

# 15. Scope Constraints

Do NOT:

- run the actual Trace2Fix investigation yet,
- invoke investigation subagents during this implementation task,
- create the INC-001 regression test,
- repair INC-001,
- add additional incidents,
- create a frontend,
- create a dashboard,
- add Docker,
- add a database,
- add CI/CD,
- add external services,
- add OpenTelemetry,
- add GitHub integration,
- perform unrelated refactoring.

Keep this phase small and focused.

---

# 16. Verification for This Task

After implementation:

1. validate the Trace2Fix skill file structure/front matter,
2. verify benchmark CLI help works,
3. test benchmark timing using a harmless temporary/sample run if needed,
4. confirm `benchmark/results.csv` can be created/written,
5. run the existing Pytest suite once,
6. confirm INC-001 still reproduces HTTP 500,
7. confirm no regression test or fix for INC-001 was created.

Repair only unintended infrastructure issues.

At the end report only:

1. files created/modified,
2. Trace2Fix skill status,
3. benchmark tool status,
4. test-suite result,
5. confirmation that INC-001 remains unresolved.

Then stop.

---

# Prompt 4 :

Perform a final **pre-investigation demo-readiness quality gate** on the existing Trace2Fix prototype.

This is a narrow verification task.

Do NOT investigate the root cause of INC-001.

Do NOT run the Trace2Fix investigation.

Do NOT spawn subagents.

Do NOT repair the seeded incident.

Do NOT create a regression test.

Do NOT redesign or extend the project.

The objective is only to verify that the environment is ready for the official Trace2Fix demonstration.

---

# 1. Verify Existing Test Suite

Run:

```bash
python -m pytest
```

Expected:

- all existing tests pass,
- zero failures.

If an unrelated infrastructure issue prevents the normal tests from passing, repair only that blocking issue.

Do NOT add coverage for INC-001.

---

# 2. Verify Development Behavior

Verify the normal development-config payment flow.

A non-base-currency payment under development configuration should return:

```text
HTTP 200
```

Confirm that normal application behavior remains functional.

---

# 3. Verify Incident Reproduction

Run:

```bash
python scripts/reproduce_incident.py
```

Expected:

```text
HTTP 500
```

Confirm that:

- INC-001 is still reproducible,
- the seeded incident remains unresolved,
- the reproducer completes normally,
- it does not print a diagnosis or root-cause explanation.

Do NOT investigate why it fails during this task.

The fact that a production-style failure exists is sufficient.

---

# 4. Verify Incident File Integrity

Inspect:

```text
incidents/INC-001.md
```

Check only for demo integrity.

It should contain:

- incident identifier,
- symptoms,
- timeline,
- impact,
- affected endpoint,
- relevant runtime context.

It must NOT reveal:

- the confirmed root cause,
- the exact missing configuration value,
- the affected source line,
- a strong root-cause hypothesis.

Do not rewrite it unless it clearly violates these requirements.

---

# 5. Verify Log Integrity

Inspect:

```text
incidents/INC-001-logs.json
```

Confirm:

- at least 40 log entries exist,
- multiple request IDs are present,
- relevant events are mixed with realistic noise,
- the incident request can be correlated,
- runtime failure evidence exists,
- the complete root cause is not directly stated.

Do not diagnose the incident from these logs.

Only validate their usefulness as investigation evidence.

---

# 6. Verify Documentation Integrity

Inspect:

```text
docs/
```

Confirm:

- documentation references real project behavior,
- documentation is sufficient for an investigator to understand intended behavior,
- documentation does not explicitly explain INC-001,
- documentation and configuration can contribute evidence independently.

Do not modify documentation unless something is clearly broken or factually inconsistent with normal application behavior.

---

# 7. Verify README Integrity

Inspect:

```text
README.md
```

Confirm it includes correct commands for:

- dependency installation,
- running tests,
- development server,
- incident reproduction,
- Trace2Fix usage,
- benchmark usage.

Also confirm README does NOT reveal the root cause of INC-001.

Repair incorrect commands only if necessary.

---

# 8. Verify AGENTS.md Integrity

Inspect:

```text
AGENTS.md
```

Confirm that it:

- provides useful repository instructions,
- prevents seeded incidents from being repaired before approval,
- explains investigation/remediation separation,
- does not reveal the root cause of INC-001.

Do not add incident-specific diagnostic knowledge.

---

# 9. Verify Trace2Fix Skill

Inspect:

```text
.bob/skills/trace2fix/SKILL.md
```

Verify only its structure and workflow.

Confirm:

- valid skill front matter,
- no hardcoded INC-001 root cause,
- `PROTOTYPE_PLAN.md` is excluded from investigation evidence,
- investigation uses exactly four independent roles:
  - Log Investigator
  - Code Investigator
  - Documentation + Configuration Investigator
  - Test Investigator
- agents are instructed to work independently,
- parallel execution is requested where supported,
- investigation is read-only,
- evidence is required,
- observations and hypotheses are distinguished,
- root-cause report is generated by the parent agent,
- the human approval gate occurs before source/test/config modification,
- remediation requires a regression test,
- targeted and full tests are run after remediation,
- verification completeness is calculated,
- failed verification cannot be reported as VERIFIED.

Do NOT execute the Trace2Fix skill during this task.

Do NOT spawn its investigators.

If the skill has a structural blocker that would prevent the intended workflow, make only the smallest correction required.

---

# 10. Verify Benchmark Tool

Run help or harmless commands against:

```text
benchmark/benchmark.py
```

Confirm support for:

- start
- stop
- record
- compare
- list

Confirm structured results can be persisted to:

```text
benchmark/results.csv
```

Do not add fake benchmark measurements for INC-001.

If testing requires temporary sample data, remove it afterward.

---

# 11. Verify Report Destination

Confirm the Trace2Fix skill has a consistent final report destination.

The current project may use:

```text
reports/<INCIDENT-ID>.md
```

That is acceptable.

Do NOT move directories simply to match an earlier plan.

Only ensure that:

- the skill,
- README,
- and report directory

agree on the final location.

---

# 12. Verify Demo Isolation

The official investigation must not obtain the answer from build-time planning artifacts.

Confirm that:

```text
PROTOTYPE_PLAN.md
```

is explicitly excluded from Trace2Fix investigation evidence.

Do not delete it in this task.

Also verify that no application comments, README section, AGENTS.md section, or skill instructions directly disclose the root cause.

---

# 13. Scope Restrictions

Do NOT:

- fix INC-001,
- diagnose INC-001,
- create its regression test,
- spawn investigation subagents,
- add additional incidents,
- add dependencies,
- build a dashboard,
- add Docker,
- add a database,
- add CI/CD,
- add integrations,
- refactor working code,
- add optional features.

This is only a readiness check.

---

# Final Output

At the end, return only:

## Demo Readiness

### Tests
PASS / FAIL

### Development Flow
PASS / FAIL

### Incident Reproduction
PASS / FAIL

### Incident Data Integrity
PASS / FAIL

### Trace2Fix Skill
PASS / FAIL

### Benchmark Tooling
PASS / FAIL

### Demo Isolation
PASS / FAIL

### Changes Made
List only blocking corrections made during this task.

### Final Status

Output exactly one of:

```text
READY FOR TRACE2FIX DEMO
```

or

```text
NOT READY — BLOCKERS REMAIN
```

Then stop.

---

# Prompt 5:

---

Use the project-specific `trace2fix` skill to investigate production incident `INC-001`.

This is the official Trace2Fix benchmark run.

## Incident Input

Incident description:

`incidents/INC-001.md`

Relevant runtime evidence:

`incidents/INC-001-logs.json`

Repository guidance:

`AGENTS.md`

Follow the Trace2Fix skill strictly.

---

# Benchmark Requirement

This run will be compared against a previously recorded manual debugging baseline.

Do not perform unnecessary exploration or unrelated work.

The objective is to determine the root cause using the Trace2Fix multi-agent workflow while minimizing direct developer effort.

---

# Investigation Only

Perform the **investigation phase only**.

Do NOT:

- modify application source,
- modify configuration,
- modify documentation,
- create or modify tests,
- implement a fix,
- perform remediation,
- read build-time planning material,
- read `PROTOTYPE_PLAN.md`,
- assume prior knowledge of INC-001.

Treat the incident as unknown.

---

# Launch Four Independent Investigators

Spawn exactly four focused read-only investigation subagents.

Where supported, launch them in parallel.

## 1. Log Investigator

Scope primarily:

`incidents/INC-001-logs.json`

Responsibilities:

- identify the relevant request,
- correlate request IDs,
- reconstruct the runtime sequence,
- identify the exception and failure stage,
- report runtime observations.

Do not inspect other investigators' conclusions.

Return only:

```text
AGENT: Log Investigator

OBSERVATIONS
...

EVIDENCE
...

HYPOTHESES
...

UNKNOWNS
...
```

---

## 2. Code Investigator

Scope primarily:

`app/`

Responsibilities:

- identify the affected endpoint,
- trace the relevant execution path,
- identify the runtime failure point,
- identify assumptions made by the implementation,
- report affected components.

Do not rely on conclusions from other investigators.

Return:

```text
AGENT: Code Investigator

EXECUTION PATH
...

OBSERVATIONS
...

EVIDENCE
...

HYPOTHESES
...

UNKNOWNS
...
```

---

## 3. Documentation + Configuration Investigator

Scope primarily:

`docs/`

and:

`config/`

Responsibilities:

- determine intended application/configuration behavior,
- inspect relevant environment configuration,
- identify meaningful mismatches,
- determine whether documentation and actual configuration agree.

Do not modify files.

Return:

```text
AGENT: Documentation + Configuration Investigator

EXPECTED BEHAVIOR
...

OBSERVATIONS
...

EVIDENCE
...

MISMATCHES
...

HYPOTHESES
...

UNKNOWNS
...
```

---

## 4. Test Investigator

Scope primarily:

`tests/`

Responsibilities:

- identify relevant existing tests,
- determine what payment scenarios are currently covered,
- identify missing edge-case coverage related to the observed failure,
- recommend a regression-test scenario.

Do NOT create the test yet.

Return:

```text
AGENT: Test Investigator

EXISTING COVERAGE
...

MISSING COVERAGE
...

EVIDENCE
...

RECOMMENDED REGRESSION TEST
...

UNKNOWNS
...
```

---

# Independence Requirement

The four investigators must complete their own evidence gathering before the parent agent synthesizes their findings.

One investigator should not be given another investigator's diagnosis before completing its own analysis.

Use read-only/exploration subagents.

Do not spawn additional agents unless absolutely required.

---

# Parent-Agent Synthesis

After all four investigators return, correlate their findings.

Explicitly distinguish:

## Observed Facts

Claims directly supported by repository evidence.

## Hypotheses

Possible explanations that are not yet sufficiently supported.

## Correlated Root Cause

The conclusion supported by the combined evidence.

For meaningful conclusions, reference concrete repository files and relevant locations.

Identify any conflicting evidence rather than hiding it.

---

# Produce the Trace2Fix Root Cause Report

Output:

```text
TRACE2FIX ROOT CAUSE REPORT
```

with these sections:

```text
Incident

Observed Symptoms

Runtime Sequence

Execution Path

Observed Facts

Root Cause

Supporting Evidence

Affected Components

Existing Test Gap

Recommended Regression Test

Proposed Remediation

Confidence

Remaining Unknowns
```

Use confidence:

- HIGH
- MEDIUM
- LOW

Use HIGH only if multiple independent evidence sources support the conclusion.

---

# Mandatory Stop

After producing the root-cause report, output:

```text
STATUS: AWAITING REMEDIATION APPROVAL
```

Then STOP.

Do not:

- create the regression test,
- modify files,
- run remediation,
- fix the incident.

Wait for the exact explicit developer approval before continuing:

`Root cause approved. Proceed with remediation.`
