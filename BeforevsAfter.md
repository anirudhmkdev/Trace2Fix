manual baseline:
- Investigation: 49.4s
- Resolution: 66.3s
- Total: 115.7s
- Manual steps: 19
- Root cause: correct
- Fix attempts: 1
- Regression test: created
- Verification: 10/10

Trace2fix (Agent):
Log Investigator: ~11s
Test Investigator: ~14s
Docs + Config Investigator: ~15s
Code Investigator: ~18s

---

| Metric | Manual | Trace2Fix |
|---|---:|---:|
| Manual developer steps | 19 | 4 |
| Root cause correct | Yes | Yes |
| Fix attempts | 1 | 1 |
| Regression test created | Yes | Yes |
| Verification completeness | 10/10 | 10/10 |
| Resolution wall time | 438.3s | 334.1s |
| Parallel investigation | No | 4 agents |
| Evidence synthesis | Manual | Automated |
| Incident report | Manual/not automatic | Automatically generated |
