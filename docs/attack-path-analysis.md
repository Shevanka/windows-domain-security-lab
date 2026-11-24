# Attack Path Analysis

This document describes how the threat scenarios in the [threat model](threat-model.md)
could be chained into realistic attack paths inside `corp.lab`, and which control
breaks each step. It is a **design-level** analysis: it reasons about the planned
configuration and does not contain exploit instructions.

---

## Path A: Password Guessing to Domain Workstation Access

```text
External / internal attacker
   │  (1) Guess passwords for known usernames           ← T-01, T-02
   ▼
Valid standard user credentials
   │  (2) Interactive logon to CLIENT01
   ▼
Standard user session on CLIENT01
```

| Step | Enabling weakness | Breaking control | Detection |
| :--- | :--- | :--- | :--- |
| 1 | Weak or predictable password | 14-character minimum, complexity, lockout after 5 attempts | 4625, 4740, 4771 on DC01 |
| 2 | Valid credentials obtained | — (legitimate logon) | 4624 logon type 2/10 outside expected hours |

**Residual risk:** lockout can be abused for denial of service, and a correctly guessed password produces only a normal-looking logon event.

---

## Path B: Workstation Compromise to Domain Admin

```text
Malware running as standard user on CLIENT01
   │  (1) Attempt to become local administrator         ← T-03
   ▼
Local administrator on CLIENT01
   │  (2) Harvest cached credentials of admins who logged on to CLIENT01
   ▼
Domain administrator credentials
   │  (3) Add attacker account to Domain Admins         ← T-04
   ▼
Domain-wide control (GPO edits, log deletion)           ← T-05, T-06
```

| Step | Enabling weakness | Breaking control | Detection |
| :--- | :--- | :--- | :--- |
| 1 | User already local admin, or misconfigured GPO | Restricted Groups on Lab-Workstations | 4732 on CLIENT01, 4688 for suspicious tools |
| 2 | Domain admin logged on interactively to a workstation | Separate `adm-` accounts; admins do not log on to CLIENT01 | 4624 + 4672 for privileged accounts on CLIENT01 |
| 3 | Compromised privileged credentials | Minimal Domain Admins membership | 4728 / 4756 on DC01 |
| 4 | Domain admin can edit GPOs and clear logs | Event forwarding and review | 5136 (GPO change), 1102 (log cleared), 4719 |

**Key insight:** the most important control in this path is **not logging on to the
workstation with a domain admin account**. Without it, steps 2–4 collapse into one move.

---

## Path C: Excessive Group Membership to Data Exposure

```text
User with excessive privileges / incorrect group membership   ← T-07
   │  (1) Browse \\FS01\<department share>
   ▼
Read or modify another department's data
```

| Step | Enabling weakness | Breaking control | Detection |
| :--- | :--- | :--- | :--- |
| 1 | User added to the wrong `GG-Share-*` group | Group-based NTFS permissions, periodic membership review | 5140 / 5145 on FS01, 4728 when membership changed |

---

## Summary: Control Coverage per Path

| Control | Path A | Path B | Path C |
| :--- | :---: | :---: | :---: |
| Password + lockout policy | ✅ | | |
| Restricted Groups | | ✅ | |
| Separate admin accounts | | ✅ | |
| Privileged group auditing | | ✅ | ✅ |
| Directory Service Changes auditing | | ✅ | |
| Event forwarding | ✅ | ✅ | ✅ |
| Group-based share permissions | | | ✅ |

Each path is validated with the test cases in [test-results.md](test-results.md).
