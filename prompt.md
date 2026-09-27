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
