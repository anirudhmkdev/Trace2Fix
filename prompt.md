Implement the prototype foundation described in `PROTOTYPE_PLAN.md`.

This task should build the complete sample application and incident environment.

Do NOT implement the Trace2Fix Bob skill yet.

Do NOT fix INC-001.

## Important demo-integrity rules

`PROTOTYPE_PLAN.md` contains implementation knowledge about the seeded incident because it is being used during construction.

However, the actual runtime project artifacts must NOT directly reveal the root cause.

Therefore:

1. `config/prod.yaml` must simply omit the relevant configuration value.
   - Do NOT add comments explaining that a value is intentionally missing.
   - Do NOT write comments such as `seeded bug`, `intentional defect`, or similar.

2. Application source code must look like normal application code.
   - Do NOT add comments identifying intentionally unsafe code.
   - Do NOT mention INC-001 inside application source.
   - Do NOT explain the hidden defect in comments or docstrings.

3. `incidents/INC-001.md` must contain only:
   - incident ID
   - timeline
   - affected endpoint
   - HTTP status
   - request ID
   - user/customer impact
   - observed symptoms
   - location of relevant logs

   It must NOT mention:
   - `EXCHANGE_RATE`
   - missing configuration
   - configuration as a suspected cause
   - the affected source line
   - the final root cause

4. `incidents/INC-001-logs.json` must provide useful runtime evidence without explaining the cause.

5. `README.md` may state that the repository contains a seeded production incident for Trace2Fix demonstration purposes, but it must NOT reveal what causes INC-001.

6. `AGENTS.md`, if created during this task, may state:
   `Do not repair seeded incidents unless explicitly requested during remediation.`

   It must NOT describe the root cause of INC-001.

These restrictions are important because the future Trace2Fix agents must discover the problem through evidence correlation rather than reading an answer embedded in the repository.

---

# Build Requirements

Implement the repository described in `PROTOTYPE_PLAN.md`.

Keep the application intentionally small.

## 1. Dependencies

Create `requirements.txt` with only the dependencies needed for:

- FastAPI
- Uvicorn
- PyYAML
- Pydantic
- Pytest
- HTTPX / FastAPI TestClient support

Do not add unnecessary packages.

---

## 2. Application

Implement:

`app/__init__.py`

`app/main.py`

`app/config.py`

`app/payment.py`

`app/models.py`

Required API behavior:

### Health endpoint

Add a simple health endpoint such as:

`GET /health`

Expected response:

HTTP 200.

### Order endpoint

Implement the minimal order endpoint described in the plan.

Do not add persistence or a database.

### Payment endpoint

Implement:

`POST /payments`

It should accept approximately:

- order ID
- amount
- currency

For normal development configuration:

- USD/basic payment behavior should work.
- currency-conversion payment should work.
- response should return HTTP 200.

---

# 3. Configuration

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

# 4. Payment / Currency Conversion

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

# 5. Existing Tests

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

# 6. Incident

Create:

`incidents/INC-001.md`

Make it realistic but concise.

Use a request ID such as:

`req-1842`

Describe something equivalent to:

- customer payment request failed
- POST /payments
- HTTP 500
- observed production time window
- payment processing affected
- investigation required

Do not reveal the cause.

---

# 7. Incident Logs

Create:

`incidents/INC-001-logs.json`

Requirements:

- at least 40 structured log entries
- realistic noise from normal application activity
- multiple request IDs
- the relevant request ID appears several times
- payment processing begins
- currency-conversion stage appears
- runtime TypeError appears
- HTTP 500 appears
- relevant evidence is not the first log entry
- logs do not say that configuration is missing
- logs do not name the root cause

The log investigator should need to filter/correlate events.

---

# 8. Documentation

Create:

`docs/architecture.md`

`docs/payment-flow.md`

`docs/api.md`

`docs/configuration.md`

Keep each document short.

Documentation must reference real project behavior.

The documents should collectively explain:

- application structure
- payment execution path
- API contract
- expected configuration

`docs/configuration.md` should document required configuration normally, including the configuration expected by currency conversion.

Do NOT say that production is missing it.

The documentation should describe intended behavior, not explain INC-001.

---

# 9. Deterministic Reproduction Script

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
   - request being executed
   - HTTP status
   - response

Expected result:

HTTP 500.

The script must not print:

- missing configuration
- `EXCHANGE_RATE is missing`
- root-cause explanations

Its purpose is reproduction, not diagnosis.

Ensure expected server exception behavior is handled so the script can complete and clearly display the simulated HTTP 500 rather than simply terminating with an uncaught Python traceback.

---

# 10. Benchmark Foundation

Create:

`benchmark/benchmark.py`

`benchmark/results.md`

For this phase, implement the simple benchmark functionality described in `PROTOTYPE_PLAN.md`.

At minimum:

- start timer
- stop timer
- investigator/workflow name
- incident ID
- elapsed seconds

Keep it simple.

More detailed Trace2Fix metrics will be added later.

---

# 11. README

Create `README.md`.

Include:

- project purpose at a high level
- installation
- normal test command
- development server command
- production server command
- reproduction script command
- optional curl example
- statement that INC-001 is a seeded production incident used for the Trace2Fix workflow

Do NOT disclose the root cause of INC-001.

A reader should be able to reproduce the failure without knowing why it occurs.

---

# Verification

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

- `INC-001.md`
- `README.md`
- `INC-001-logs.json`
- actual source comments
- actual configuration comments

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

# Constraints

Do NOT:

- implement Trace2Fix skill yet
- fix INC-001
- create the missing regression test
- create additional incidents
- add a database
- add Docker
- build frontend/UI
- create CI/CD
- add OpenTelemetry
- add external APIs
- create microservices
- add authentication
- perform unrelated refactoring
- use subagents for this implementation task

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
