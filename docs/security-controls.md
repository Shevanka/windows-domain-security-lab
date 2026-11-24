# Security Controls

This document maps each threat scenario in the [threat model](threat-model.md) to the
controls planned for the `corp.lab` domain. Controls are grouped by type and are
implemented mainly through Group Policy Objects (GPOs) linked to dedicated
Organizational Units (OUs).

---

## 1. OU and GPO Structure

```text
corp.lab
├── Domain Controllers          ← Default Domain Controllers Policy + SEC-DC-Audit
├── Lab-Workstations            ← SEC-WS-Hardening, SEC-WS-RestrictedGroups, SEC-WS-Audit
├── Lab-Servers                 ← SEC-SRV-Audit (FS01)
├── Lab-Users                   ← standard user accounts
└── Lab-Admins                  ← separate administrative accounts
```

| GPO | Linked to | Purpose |
| :--- | :--- | :--- |
| Default Domain Policy | Domain root | Password and account lockout policy |
| SEC-DC-Audit | Domain Controllers | Advanced audit policy for authentication and directory changes |
| SEC-WS-Hardening | Lab-Workstations | Endpoint hardening baseline |
| SEC-WS-RestrictedGroups | Lab-Workstations | Controls membership of the local Administrators group |
| SEC-WS-Audit | Lab-Workstations | Logon, process creation and policy-change auditing |
| SEC-SRV-Audit | Lab-Servers | File share access auditing on FS01 |

---

## 2. Control Mapping

| Threat | Control | Type | Implementation |
| :--- | :--- | :--- | :--- |
| **T-01** Password guessing | Account lockout policy | Preventive | Default Domain Policy → Account Lockout Policy |
| **T-01** Password guessing | Failed logon auditing | Detective | Audit Credential Validation + Logon (Failure) |
| **T-02** Weak passwords | Password policy | Preventive | Minimum length, complexity, history |
| **T-03** Local admin escalation | Restricted Groups | Preventive | SEC-WS-RestrictedGroups defines local Administrators |
| **T-03** Local admin escalation | Group membership auditing | Detective | Audit Security Group Management (event 4732) |
| **T-04** Privileged group change | Separate admin accounts | Preventive | Daily accounts never belong to Domain Admins |
| **T-04** Privileged group change | Group change alerts | Detective | Events 4728 / 4732 / 4756 reviewed on DC01 |
| **T-05** Log tampering | Log size and retention | Preventive | Security log size increased, overwrite as needed |
| **T-05** Log tampering | Log-clear detection + forwarding | Detective | Event 1102 and Windows Event Forwarding to DC01 |
| **T-06** GPO modification | Least privilege on GPOs | Preventive | Only Domain Admins can edit; delegation reviewed |
| **T-06** GPO modification | Directory change auditing | Detective | Audit Directory Service Changes (event 5136) |
| **T-07** Share access | Group-based NTFS + share permissions | Preventive | One security group per department share |
| **T-07** Share access | File share auditing | Detective | Audit File Share / Detailed File Share (5140 / 5145) |

---

## 3. Account Policies (Default Domain Policy)

| Setting | Value | Reason |
| :--- | :--- | :--- |
| Minimum password length | 14 characters | Resists guessing; passphrase-friendly |
| Password must meet complexity requirements | Enabled | Blocks trivial passwords |
| Enforce password history | 24 passwords | Prevents immediate reuse |
| Maximum password age | 365 days | Long-lived passphrases instead of forced frequent rotation |
| Minimum password age | 1 day | Prevents cycling through history |
| Store passwords using reversible encryption | Disabled | Keeps hashes non-reversible |
| Account lockout threshold | 5 invalid attempts | Slows online guessing (T-01) |
| Account lockout duration | 15 minutes | Limits lockout-based denial of service |
| Reset account lockout counter after | 15 minutes | Matches lockout duration |

---

## 4. Privileged Access

- Each administrator uses two accounts: a standard account for daily work and an
  `adm-` account stored in `Lab-Admins` for administration only.
- `Domain Admins`, `Enterprise Admins` and `Schema Admins` membership is kept to the
  minimum required and reviewed after every lab session.
- The built-in `Administrator` account is renamed and given a long random password
  kept outside the lab.
- Standard users are not members of the local `Administrators` group on CLIENT01
  (enforced by Restricted Groups).

---

## 5. Logging and Monitoring

- Advanced Audit Policy is used (see [audit-policy.md](audit-policy.md)), with
  *"Force audit policy subcategory settings to override audit policy category settings"* enabled.
- Security log maximum size is raised so that a full lab session fits in the log.
- CLIENT01 and FS01 forward selected security events to DC01 through Windows Event
  Forwarding (WEF).
- Events are reviewed using the queries in [event-log-analysis.md](event-log-analysis.md).

---

## 6. Known Gaps (First Iteration)

These are accepted limitations documented in [network-plan.md](../architecture/network-plan.md):

- No network segmentation or dedicated firewall between workstation and server zones.
- No IDS/EDR; detection relies on native Windows event logs.
- Single domain controller, so no redundancy.
- The event collector runs on DC01, so a domain admin compromise also exposes the logs (residual risk of T-05).
