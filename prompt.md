# Trace2Fix — IBM Bob 40-Bobcoin Build Strategy

## Objective

Build the complete Trace2Fix hackathon prototype using IBM Bob while minimizing Bobcoin consumption.

The final prototype must demonstrate:

- FastAPI sample application
- realistic seeded production incident
- logs, configuration, source code, documentation, and tests
- reusable Trace2Fix Bob skill
- Agent mode
- four investigative subagents
- parallel/independent investigation
- evidence-backed root-cause analysis
- human approval gate
- regression-test creation
- minimal fix
- automated verification
- Trace2Fix incident report
- before/after metrics

Do **not** build a dashboard unless substantial Bobcoins remain after the complete workflow works.

---

# Budget Strategy

Treat these as budget guardrails, not guaranteed Bobcoin costs.

| Stage | Desired balance after stage |
|---|---:|
| Start | 40 |
| Planning | ≥36 |
| Sample project complete | ≥28 |
| Trace2Fix skill + tooling complete | ≥20 |
| Quality gate complete | ≥16 |
| Live investigation complete | ≥10 |
| Remediation complete | ≥5 |
| Final reserve | 3–5 |

If consumption is higher than expected, skip optional work immediately.

The priority is:

```text
WORKING END-TO-END DEMO
        >
EXTRA INCIDENTS
        >
DASHBOARD
        >
VISUAL POLISH
```

---

# Important Bob Usage Rules

During project construction:

- Use Plan mode once.
- Use Agent mode for implementation.
- Avoid Ask mode unless absolutely necessary.
- Do not ask Bob to explain code you can inspect yourself.
- Do not repeatedly ask Bob to summarize the repository.
- Do not use subagents during ordinary project construction.
- Save subagents for the Trace2Fix investigation demonstration.
- Run multiple related implementation tasks in one prompt.
- Tell Bob explicitly not to perform unnecessary refactoring.
- Tell Bob to stop when acceptance criteria are satisfied.
- Create repository context early.
- Start a new Bob task between major phases.
- Keep the investigation and remediation in the same Bob task so Bob retains incident context.
- Do not request a dashboard until the complete debugging workflow works.

---

# PHASE 0 — Manual Setup

## Bobcoins

0.

Do this yourself before prompting Bob.

Create/open an empty directory:

```text
trace2fix-demo/
```

Initialize Git if desired:

```bash
git init
```

Open the directory in IBM Bob IDE.

Do not manually create the application.

The purpose is for the repository implementation to clearly demonstrate Bob involvement.

---

# PHASE 1 — Architecture Plan

## Mode

Plan Mode

## Goal

Have Bob create one concise implementation plan.

Do not let it start coding yet.

## Prompt 1

```text
We are building a hackathon prototype called Trace2Fix.

Trace2Fix demonstrates an agentic production-debugging workflow powered by IBM Bob 2.0.

The prototype must be intentionally small and optimized for minimal implementation effort.

P0 requirements:

1. Build a Python FastAPI payment/order sample application.
2. Use Pytest.
3. Use YAML production/development configuration.
4. Create one deterministic seeded production bug:
   EXCHANGE_RATE is missing from production configuration and payment/currency code assumes it exists, causing a payment request to fail with HTTP 500.
5. Normal existing tests must pass because they test normal configured behavior.
6. Create a realistic incident file and noisy structured log for INC-001.
7. Create minimal architecture, payment-flow, API and configuration documentation.
8. Later we will create a Bob skill called trace2fix.
9. Trace2Fix will investigate an incident using four independent investigation roles:
   - logs
   - source code
   - documentation/configuration
   - existing tests
10. After investigation, the parent agent must synthesize evidence and STOP for human approval.
11. After approval it will create a regression test, demonstrate the test failing against the bug, implement the smallest safe fix, run targeted tests and the full suite, check docs/configuration, and generate a report.
12. We need simple benchmark tooling to record manual vs Trace2Fix investigation/resolution time and verification completeness.

Constraints:
- Hackathon prototype, not production software.
- Prefer the fewest files and dependencies possible.
- No database unless necessary.
- No frontend.
- No dashboard.
- No Docker unless absolutely required.
- No GitHub integration.
- No OpenTelemetry.
- No extra microservices.
- No unnecessary abstractions.
- Do not use subagents for this planning task.
- Do not implement anything yet.

Create a concise implementation plan in PROTOTYPE_PLAN.md.

Include:
- repository structure
- implementation order
- primary incident behavior
- acceptance criteria
- commands for running the app/tests
- what must remain intentionally broken before Trace2Fix investigates INC-001

Keep PROTOTYPE_PLAN.md concise, preferably under 150 lines.

Stop after writing the plan.
```

## Do not ask follow-up questions

Read `PROTOTYPE_PLAN.md` yourself.

If it looks reasonable, continue.

Do not spend another Bob request saying:

> Looks good.

Simply start a new task.

---

# PHASE 2 — Build the Complete Sample Application

## Mode

Agent Mode

## Goal

Build almost all non-agentic infrastructure in one Bob task.

This should be one of your largest build prompts.

## Prompt 2

```text
Implement the P0 sample project described in PROTOTYPE_PLAN.md.

Complete the entire foundation in this task.

Required repository structure should remain minimal but include:

app/
tests/
config/
docs/
logs/
incidents/
scripts/
trace2fix/reports/
trace2fix/metrics/

Build a small FastAPI order/payment application.

Required behavior:

- Provide a health endpoint.
- Provide an order payment endpoint such as POST /orders/{order_id}/pay.
- Payment processing must call a currency conversion function.
- Development configuration contains a valid EXCHANGE_RATE.
- Production configuration intentionally omits EXCHANGE_RATE.
- The seeded bug is that application logic does not safely handle the missing exchange rate.
- Triggering INC-001 using production configuration must result in the intended failure.
- Normal existing tests use valid configuration and must pass.

Create:
- app source files
- requirements.txt
- config/development.yaml
- config/production.yaml
- tests covering normal behavior
- incidents/INC-001.md
- logs/incident-001.log
- docs/architecture.md
- docs/payment-flow.md
- docs/api.md
- docs/configuration.md
- scripts/reproduce_incident.py
- README.md

INC-001 should only describe symptoms and relevant evidence locations. It must NOT reveal the root cause.

The log should contain realistic noise plus:
- request ID
- payment request
- currency conversion activity
- final exception
- HTTP 500

Documentation should state intended behavior and enough configuration information for a documentation investigator to contribute evidence.

IMPORTANT:
The seeded INC-001 defect must remain unfixed.

Do NOT:
- build Trace2Fix yet
- create a frontend
- create a dashboard
- add Docker
- add CI/CD
- add a database unless required
- create additional incidents
- refactor beyond requirements
- use subagents

Run the existing test suite when finished.

Then run the reproduction script and confirm:
1. normal tests pass
2. INC-001 production reproduction demonstrates the intended failure

Repair only unintended implementation errors.

Do not repair the intentional INC-001 defect.

Stop once these acceptance criteria pass.
```

## Expected result

You should now have a real debugging target.

Before continuing, manually verify:

```bash
pytest
```

and:

```bash
python scripts/reproduce_incident.py
```

No Bob prompt is required to verify something you can see yourself.

---

# PHASE 2.5 — Initialize Persistent Context

Once the repository exists, use Bob's initialization command:

```text
/init
```

Allow Bob to create/update `AGENTS.md`.

This is worth spending some budget because future tasks can use persistent repository context rather than repeatedly rediscovering the project.

After `/init`, manually inspect `AGENTS.md`.

Make sure it clearly mentions:

```text
INC-001 is intentionally broken until the Trace2Fix remediation stage.

Do not silently fix seeded incidents during unrelated work.

Trace2Fix investigations must not modify application source before the human approval gate.

Prefer minimal changes and avoid unrelated refactoring.
```

If those four rules are absent, it is cheaper to edit `AGENTS.md` yourself than spend another Bob interaction asking it to add four sentences.

---

# PHASE 3 — Build the Trace2Fix Skill and Benchmark Tools

## Mode

Agent Mode

Start a new Bob task.

## Prompt 3

```text
Implement the Trace2Fix workflow infrastructure for this repository.

Do NOT fix INC-001.

Create a project-specific IBM Bob skill at:

.bob/skills/trace2fix/SKILL.md

Use valid YAML front matter including:
name: trace2fix
description: Investigate production incidents through independent log, code, documentation/configuration, and test analysis, then perform evidence-backed remediation after human approval.
user-invocable: true

The Trace2Fix skill must implement this workflow:

INVESTIGATION PHASE

1. Read the requested incident.
2. Do not modify application/source/config/test files.
3. Spawn exactly four focused read-only/explore investigation subagents when an incident investigation is requested:
   A. Log Investigator
   B. Code Investigator
   C. Documentation + Configuration Investigator
   D. Test Investigator
4. Investigators should work independently so one investigator's hypothesis does not bias another.
5. Run independent investigations in parallel where Bob supports it.
6. Each investigator returns only a concise structured summary containing:
   - observations
   - evidence with file references
   - hypotheses
   - unknowns
7. Parent agent correlates all findings.
8. Parent distinguishes evidence from hypothesis.
9. Parent produces TRACE2FIX ROOT CAUSE REPORT containing:
   - incident
   - symptoms
   - execution path
   - root cause
   - evidence
   - affected files
   - missing test
   - proposed remediation
   - confidence
   - remaining unknowns
10. STOP before editing files.
11. Require explicit developer approval.

REMEDIATION PHASE

Only after explicit approval:

1. Create the smallest regression test that reproduces the confirmed bug.
2. Run that test against the buggy implementation and record that it fails.
3. Apply the smallest reasonable source/configuration fix.
4. Run the regression test again.
5. Run relevant subsystem tests.
6. Run the complete Pytest suite.
7. Review changed files for unrelated modifications.
8. Check documentation impact.
9. Check configuration impact.
10. Calculate verification completeness using:
   - root cause evidence present
   - defect reproduced
   - regression test created
   - regression test passes
   - relevant tests pass
   - full suite passes
   - documentation checked
   - configuration checked
   - changed files reviewed
   - no unrelated modifications
11. Write trace2fix/reports/<INCIDENT-ID>.md.
12. Never claim VERIFIED when any required verification check fails.

Keep SKILL.md focused. Create a supporting checklist/template file beside the skill only if it materially reduces the main skill size.

Also create lightweight benchmark tooling:

scripts/benchmark.py

It should allow a human to record:
- workflow = manual or trace2fix
- incident ID
- investigation start/end
- resolution start/end or total time
- manual developer steps
- fix attempts
- root cause correct yes/no
- regression test created yes/no
- verification completed
- verification total

Persist results to:

trace2fix/metrics/results.csv

Keep the implementation simple and standard-library based where possible.

Add minimal usage instructions to README.md.

Do NOT:
- investigate INC-001 now
- fix INC-001
- create a dashboard
- create more incidents
- use subagents during this implementation task
- add unnecessary dependencies

Run any relevant tests or syntax validation.

Stop as soon as the skill and benchmark tooling satisfy these requirements.
```

## Expected files

You should now have approximately:

```text
.bob/skills/trace2fix/SKILL.md
scripts/benchmark.py
trace2fix/metrics/results.csv
```

plus perhaps one small skill-supporting template/checklist.

---

# PHASE 4 — One Quality-Gate Task

## Mode

Agent Mode

Start a new Bob task.

This is your last implementation task before running Trace2Fix itself.

## Prompt 4

```text
Perform a narrow demo-readiness quality gate on the existing Trace2Fix prototype.

IMPORTANT:
INC-001 is intentionally broken.
Do NOT diagnose or fix its root cause.

Only verify project mechanics.

Check:

1. Dependencies install correctly.
2. FastAPI project imports correctly.
3. Existing normal tests pass.
4. scripts/reproduce_incident.py successfully reproduces the intended INC-001 failure.
5. incidents/INC-001.md does not reveal the root cause.
6. logs/incident-001.log contains sufficient evidence plus realistic noise.
7. docs contain useful but non-spoiling information.
8. .bob/skills/trace2fix/SKILL.md has valid front matter and references paths that actually exist.
9. scripts/benchmark.py works.
10. README commands are correct.

Fix ONLY unintended blocking issues discovered by these checks.

Do not:
- repair INC-001
- redesign anything
- refactor working code
- add features
- add dependencies unless absolutely necessary
- create a dashboard
- create additional incidents
- use subagents

Run the minimum commands required to verify the above.

At the end output only:
- checks passed
- blocking issues repaired
- remaining intentional failure

Then stop.
```

At this point your software-building work should largely be over.

---

# PHASE 5 — Record the Manual Baseline

## Bobcoins

0 Bobcoins if you do it yourself.

This is important.

Do **not** use Bob for the manual baseline because the baseline represents the conventional developer workflow.

Before running Trace2Fix, record a baseline.

Use:

```bash
python scripts/benchmark.py ...
```

according to the implementation Bob created.

Measure:

```text
Manual investigation time
Manual total resolution time
Manual developer actions
Fix attempts
Verification checks completed
```

For the cleanest experiment, reset the repository afterward:

```bash
git restore .
```

or return to your clean pre-investigation commit.

The official live Trace2Fix demonstration should start with the original intentional bug.

---

# PHASE 6 — The Main IBM Bob Demonstration

This is the most important Bob session in the entire project.

Do not waste the remaining budget before reaching this phase.

## Mode

Agent Mode

Start a **new Bob task**.

Do not start another new task until both investigation and remediation are finished.

Use/select the `trace2fix` skill.

## Prompt 5 — Investigation

```text
Use the Trace2Fix skill to investigate INC-001.

Incident:
incidents/INC-001.md

Relevant production evidence:
logs/incident-001.log

This is the official Trace2Fix demonstration.

Follow the skill strictly.

INVESTIGATION ONLY.

Spawn exactly four independent read-only investigation subagents:

1. Log Investigator
   Analyze runtime logs, request IDs, error sequence and exceptions.

2. Code Investigator
   Trace the relevant endpoint and execution path through source code.

3. Documentation + Configuration Investigator
   Determine intended behavior and inspect relevant project documentation and configuration.

4. Test Investigator
   Inspect existing coverage and identify the missing regression scenario.

Use explore/read-only subagents.

Run independent investigations in parallel where supported.

Do not allow one investigator's hypothesis to bias the others before they return their findings.

Each subagent should return a concise structured summary to the parent containing:
- observations
- repository evidence
- hypotheses
- unknowns

The parent agent must then correlate all four results.

For every important root-cause claim, identify supporting repository evidence.

Distinguish:
OBSERVED FACT
from
HYPOTHESIS.

Produce the Trace2Fix Root Cause Report.

DO NOT:
- modify source files
- modify tests
- modify configuration
- implement a fix
- create the regression test yet

End at the HUMAN APPROVAL GATE.

Wait for my explicit approval before remediation.
```

## This is where you capture evidence

Take screenshots/video showing Bob spawning the investigative agents.

This is central to your hackathon story.

Bob's documentation says subagents run in independent contexts and return focused summaries to the parent, which is exactly what you are demonstrating.

---

# PHASE 7 — Approve Remediation

Stay in the **same Bob task**.

Do not start a new conversation because Bob already has the investigation evidence.

## Prompt 6

```text
ROOT CAUSE APPROVED.

Proceed with the Trace2Fix remediation phase for INC-001.

Follow the existing Trace2Fix skill and the approved root-cause report.

Required sequence:

1. Create ONE minimal regression test that specifically reproduces the confirmed INC-001 defect.

2. Before modifying the application implementation, run that regression test against the buggy code.

3. Capture and report that expected failing result.

4. Apply the smallest safe remediation necessary to address the approved root cause.

5. Do not perform unrelated refactoring.

6. Run the new regression test again.

7. Run relevant payment/currency tests.

8. Run the complete Pytest suite.

9. Inspect the final diff and ensure there are no unrelated source modifications.

10. Check whether documentation requires an update.

11. Check whether configuration requires an update.

12. Calculate verification completeness from the Trace2Fix 10-check checklist.

13. Generate:

trace2fix/reports/INC-001.md

The report must include:
- incident summary
- observed symptoms
- root cause
- evidence
- affected components
- regression test
- before-fix test result
- remediation
- files changed
- targeted test results
- full-suite result
- documentation impact
- configuration impact
- verification checklist
- verification completeness
- remaining risks

Only mark the incident VERIFIED if all required checks actually succeed.

Do not:
- add new product features
- build UI
- redesign architecture
- create additional incidents
- make unrelated cleanup changes

Stop immediately after the report and verification are complete.
```

This completes the core prototype.

---

# PHASE 8 — Record Trace2Fix Metrics

## Bobcoins

Preferably 0.

Run your benchmark script manually and enter:

```text
Trace2Fix investigation time
Trace2Fix resolution time
Developer/manual steps
Fix attempts
Root cause correct
Regression test created
Verification checks
```

Do not spend Bobcoins asking Bob to calculate percentages you can calculate locally.

Your benchmark script should handle the simple calculations.

---

# PHASE 9 — Final Repair Task

Only run this if:

- the main demo workflow works, and
- you still have a safe Bobcoin reserve.

## Prompt 7 — Final Audit

```text
Perform a final hackathon submission audit of Trace2Fix.

Do not add features.

Check only for issues that could break the live demo or make the repository confusing to judges.

Verify:

- pytest passes
- the fixed INC-001 regression test passes
- Trace2Fix report exists
- README contains correct setup commands
- README contains exact demo steps
- .bob/skills/trace2fix/SKILL.md exists and is valid
- AGENTS.md accurately describes the repository
- metrics tooling exists
- repository does not contain accidental secrets
- no obvious generated junk or temporary files are present

Fix only blocking or clearly incorrect items.

Do not:
- redesign code
- rewrite documentation unnecessarily
- build a dashboard
- add technologies
- add incidents
- add dependencies
- use subagents

Run the required verification once.

Then stop.
```

---

# STOP HERE FOR THE CORE SUBMISSION

At this point you should have:

```text
Working FastAPI project
        ↓
Production-style seeded failure
        ↓
Incident + realistic logs
        ↓
Architecture/config docs
        ↓
Existing tests
        ↓
Trace2Fix Bob Skill
        ↓
Four investigation subagents
        ↓
Parallel/independent investigation
        ↓
Evidence-backed diagnosis
        ↓
Human approval
        ↓
Regression test failing
        ↓
Minimal fix
        ↓
Regression test passing
        ↓
Full test suite passing
        ↓
Verification checklist
        ↓
Incident report
        ↓
Before/after metrics
```

That is sufficient for a convincing Trace2Fix prototype.

---

# OPTIONAL PHASE A — Additional Seeded Incidents

Only attempt this if the primary workflow works reliably and you have substantial Bobcoins remaining.

One prompt can generate two extra benchmark incidents.

## Optional Prompt

```text
The primary Trace2Fix INC-001 workflow is complete and stable.

Create exactly TWO additional small seeded benchmark incidents to demonstrate repeatability:

INC-002:
A null-handling defect in order processing.

INC-003:
An API input-validation defect.

Requirements:

- keep each defect simple and deterministic
- each incident must have a short incident description
- each must have a noisy but useful log
- existing normal tests should still work
- do not alter or break INC-001
- do not build new infrastructure
- do not add dependencies
- do not create UI
- update README only with brief benchmark references

Keep implementation minimal.

Run the normal test suite once at the end.

Stop after creating the two incidents.
```

Do not spend more Bobcoins individually debugging INC-002 and INC-003 unless needed for your final evidence.

---

# OPTIONAL PHASE B — Dashboard

Recommendation:

**Skip it.**

A dashboard is visually attractive but adds little to the challenge compared with demonstrating Bob's actual multi-agent developer workflow.

Use:

```text
terminal
+
Bob IDE
+
generated report
+
small metrics table
```

for the hackathon presentation.

That is enough.

---

# Bobcoin Emergency Strategy

If you reach approximately half your allocation before completing the Trace2Fix skill:

Stop adding anything optional.

Your minimum viable submission becomes:

```text
FastAPI project
+ INC-001
+ logs
+ docs
+ tests
+ Trace2Fix skill
+ investigation
+ remediation
+ report
```

If you are nearing the limit before final remediation:

Do not ask Bob for:

- README rewrites
- architecture diagrams
- prettier logs
- additional tests unrelated to INC-001
- extra incidents
- dashboard
- Docker
- GitHub integration
- code cleanup
- naming improvements

Reserve Bob for the actual Trace2Fix demonstration.

---

# How to Save Bobcoins During the Build

## 1. Do not repeatedly re-explain Trace2Fix

That information belongs in:

```text
AGENTS.md
PROTOTYPE_PLAN.md
.bob/skills/trace2fix/SKILL.md
```

Let the repository carry context.

---

## 2. Use new task windows strategically

After a major implementation phase, start a fresh Bob task.

Good boundaries:

```text
Planning
    ↓ NEW TASK

Application build
    ↓ /init
    ↓ NEW TASK

Trace2Fix skill
    ↓ NEW TASK

Quality gate
    ↓ NEW TASK

Trace2Fix live demo
```

Do **not** create a new task between investigation and remediation.

---

## 3. Avoid conversational acknowledgments

Do not send messages such as:

```text
Thanks
Looks good
Continue
Explain that
Why did you do that?
Show me the file
```

unless needed.

Inspect the files directly in the IDE.

---

## 4. Bundle related work

Bad:

```text
Create payments.py.

Create tests.

Create docs.

Create config.

Create logs.

Create incident.
```

That creates multiple AI interactions.

Better:

```text
Build the complete application foundation and verify it.
```

That is why Prompt 2 deliberately covers the whole project foundation.

---

## 5. Avoid unnecessary subagents

Subagents are important for your **product demonstration**, but they are not required to build every file.

During construction:

```text
Bob Agent
    ↓
Direct implementation
```

During Trace2Fix:

```text
Bob Parent
   ├── Log subagent
   ├── Code subagent
   ├── Docs/config subagent
   └── Test subagent
```

This concentrates your agentic usage exactly where judges need to see it.

---

# Recommended Bob Sessions for Submission Evidence

Your Bob session evidence should ideally tell a clear story.

### Session 1

**Planning**

Shows Bob designing the prototype.

### Session 2

**Application implementation**

Shows Bob building the realistic debugging environment.

### Session 3

**Trace2Fix skill**

Shows Bob building the reusable agentic workflow.

### Session 4

**Quality gate**

Shows the environment works and the intentional defect remains.

### Session 5

**Trace2Fix investigation + remediation**

This is the most important session.

It demonstrates:

```text
Agent mode
Subagents
Parallel investigation
Document understanding
Code understanding
Test understanding
Evidence synthesis
Human approval
Code modification
Test creation
Command execution
Verification
```

Use this session heavily in your hackathon submission.

---

# Ideal Live Demo Sequence

Before presenting:

```text
pytest
```

shows normal project state.

Then demonstrate:

```text
python scripts/reproduce_incident.py
```

Result:

```text
HTTP 500
```

Then open:

```text
incidents/INC-001.md
```

Say:

> "A developer knows the symptom, but not the cause."

Then start Trace2Fix.

Show:

```text
            IBM Bob
               │
    ┌──────────┼───────────┐
    ↓          ↓           ↓
  Logs       Source       Docs
  Agent      Agent        Agent
               │
               ↓
             Tests
             Agent
```

Then show independent findings.

Then show:

```text
TRACE2FIX ROOT CAUSE REPORT
```

Pause at:

```text
AWAITING DEVELOPER APPROVAL
```

Approve it.

Then show:

```text
Regression test → FAIL
        ↓
Minimal fix
        ↓
Regression test → PASS
        ↓
Related tests → PASS
        ↓
Full suite → PASS
        ↓
Verification → COMPLETE
```

Finish with:

```text
Manual debugging
vs
Trace2Fix debugging
```

using your real benchmark measurements.

---

# Final Principle

Do not try to use all 40 Bobcoins.

Try to finish the complete prototype while still having a reserve.

A working:

```text
incident
→ multi-agent investigation
→ root cause
→ test
→ fix
→ verification
```

is significantly more valuable than:

```text
incident
→ partially working agents
→ dashboard
→ extra features
→ no reliable remediation
```

Build the workflow first.

Everything else is optional.
