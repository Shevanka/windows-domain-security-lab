# Test Plan and Results

Each test validates one control from [security-controls.md](security-controls.md) against
a threat from the [threat model](threat-model.md). Tests are run only inside the
isolated `CORP-LAB` network with lab test accounts.

**Status legend:** ⬜ Pending · ✅ Pass · ❌ Fail · ⚠️ Partial

> Results are filled in after each lab session. A test is marked ✅ only when both the
> **preventive** outcome and the **detective** event were observed.

---

## Summary

| ID | Test | Threat | Status | Date |
| :--- | :--- | :--- | :---: | :--- |
| TC-01 | Account lockout after repeated failed logons | T-01 | ⬜ Pending | |
| TC-02 | Weak password rejected | T-02 | ⬜ Pending | |
| TC-03 | Standard user cannot stay in local Administrators | T-03 | ⬜ Pending | |
| TC-04 | Privileged group change is logged | T-04 | ⬜ Pending | |
| TC-05 | Security log clearing is detected and forwarded | T-05 | ⬜ Pending | |
| TC-06 | GPO modification is logged | T-06 | ⬜ Pending | |
| TC-07 | Unauthorized share access is denied and logged | T-07 | ⬜ Pending | |
| TC-08 | Effective audit policy matches design | All | ⬜ Pending | |

---

## TC-01 · Account Lockout

- **Precondition:** Default Domain Policy lockout threshold = 5.
- **Steps:**
  1. On CLIENT01, attempt to log on as `testuser` with a wrong password 5 times.
  2. Attempt a 6th logon with the **correct** password.
  3. On DC01, run: `Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4740} -MaxEvents 5`
- **Expected:** 6th logon is refused while locked; events 4625 (×5) and 4740 are recorded.
- **Actual:** _pending_
- **Evidence:** _pending_

## TC-02 · Password Policy

- **Steps:** Try to set `testuser`'s password to `Password1` and to a 10-character password.
- **Expected:** Both are rejected by the domain password policy.
- **Actual:** _pending_

## TC-03 · Restricted Groups

- **Steps:**
  1. As an admin, add `testuser` to CLIENT01's local `Administrators` group.
  2. Run `gpupdate /force` on CLIENT01, then `Get-LocalGroupMember Administrators`.
- **Expected:** `testuser` is removed after policy refresh; event 4732 shows the original addition.
- **Actual:** _pending_

## TC-04 · Privileged Group Change

- **Steps:** Add a test account to `Domain Admins`, then remove it.
- **Expected:** Events 4728 (added) and 4729 (removed) on DC01 with the acting account recorded.
- **Actual:** _pending_

## TC-05 · Log Clearing Detection

- **Steps:**
  1. Confirm CLIENT01 events appear under *Forwarded Events* on DC01.
  2. Clear the Security log on CLIENT01.
- **Expected:** Event 1102 is generated and still visible on DC01 after the local log is cleared.
- **Actual:** _pending_

## TC-06 · GPO Modification

- **Steps:** Change one setting in `SEC-WS-Hardening` and save.
- **Expected:** Event 5136 on DC01 referencing the GPO object under `CN=Policies,CN=System`.
- **Actual:** _pending_

## TC-07 · Share Access

- **Steps:** As a user who is **not** in `GG-Share-Finance`, open `\\FS01\Finance`.
- **Expected:** Access denied; event 5145 (failure) on FS01.
- **Actual:** _pending_

## TC-08 · Audit Policy Verification

- **Steps:** Run `auditpol /get /category:*` on DC01, CLIENT01 and FS01.
- **Expected:** Output matches the table in [audit-policy.md](audit-policy.md).
- **Actual:** _pending_

---

Issues found during testing are recorded in [findings.md](findings.md).
