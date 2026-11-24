# Findings

This document records security issues discovered while building and testing the lab,
together with their remediation status. Each finding links to the test case that
revealed it in [test-results.md](test-results.md).

**Severity:** Critical · High · Medium · Low
**Status:** Open · Remediated · Accepted

---

## Findings Register

| ID | Title | Severity | Related threat | Test | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| _No findings recorded yet: testing in progress._ | | | | | |

---

## Design-Level Observations

These were identified during threat modelling, before hands-on testing, and are
already documented as accepted limitations of the first iteration:

| ID | Observation | Severity | Status |
| :--- | :--- | :--- | :--- |
| D-01 | Single domain controller: no redundancy for AD DS and DNS | Medium | Accepted |
| D-02 | Event collector runs on DC01, so a domain admin compromise also exposes the logs (T-05 residual risk) | Medium | Accepted |
| D-03 | No network segmentation or firewall between workstation and server zones | Medium | Accepted |
| D-04 | Detection relies only on native Windows events; no IDS/EDR | Low | Accepted |

---

## Finding Template

```markdown
### F-XX · <Short title>

- **Severity:** High
- **Related threat / test:** T-0X / TC-0X
- **Description:** What was observed.
- **Evidence:** Sanitized screenshot or event excerpt.
- **Impact:** What an attacker could achieve.
- **Remediation:** Change applied (GPO, permission, account change).
- **Retest:** Date and result.
```
