# Event Log Analysis

This guide lists the Windows Security events that matter for each threat scenario and
the PowerShell queries used to review them on DC01. Sample outputs are added after
lab sessions, **sanitized** (no passwords, hashes or real addresses outside the lab range).

---

## 1. Event Reference

| Event ID | Meaning | Logged on | Threat |
| :--- | :--- | :--- | :--- |
| 4624 | Successful logon | All | T-01 |
| 4625 | Failed logon | All | T-01 |
| 4740 | User account locked out | DC01 | T-01 |
| 4767 | User account unlocked | DC01 | T-01 |
| 4771 | Kerberos pre-authentication failed | DC01 | T-01 |
| 4672 | Special privileges assigned to new logon | All | T-04 |
| 4720 | User account created | DC01 | T-04 |
| 4728 / 4729 | Member added to / removed from a global security group | DC01 | T-04, T-07 |
| 4732 / 4733 | Member added to / removed from a local security group | DC01, CLIENT01 | T-03 |
| 4756 | Member added to a universal security group | DC01 | T-04 |
| 5136 | Directory service object modified | DC01 | T-06 |
| 4719 | System audit policy changed | All | T-05 |
| 1102 | Security audit log cleared | All | T-05 |
| 5140 | Network share accessed | FS01 | T-07 |
| 5145 | Network share object checked for access | FS01 | T-07 |
| 4688 | New process created | CLIENT01 | Malware activity |

### Useful logon types (event 4624 / 4625)

| Type | Meaning |
| :---: | :--- |
| 2 | Interactive (keyboard) |
| 3 | Network (for example SMB share) |
| 7 | Unlock |
| 10 | Remote Interactive (RDP) |

---

## 2. Review Queries

```powershell
# Failed logons in the last 24 hours, grouped by target account
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625; StartTime=(Get-Date).AddDays(-1)} |
  ForEach-Object { ([xml]$_.ToXml()).Event.EventData.Data | Where-Object Name -eq 'TargetUserName' } |
  Group-Object '#text' | Sort-Object Count -Descending

# Account lockouts
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4740} -MaxEvents 20 |
  Select-Object TimeCreated, Message

# Privileged group membership changes
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4728,4729,4732,4733,4756} -MaxEvents 50 |
  Select-Object TimeCreated, Id, Message

# GPO / directory object changes
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=5136} -MaxEvents 20 |
  Select-Object TimeCreated, Message

# Log cleared or audit policy changed (also check Forwarded Events)
Get-WinEvent -FilterHashtable @{LogName='Security','ForwardedEvents'; Id=1102,4719} -ErrorAction SilentlyContinue |
  Select-Object TimeCreated, MachineName, Id
```

---

## 3. Detection Logic

| Pattern | Suggested threshold | Response |
| :--- | :--- | :--- |
| Many 4625 for one account | ≥ 5 within 15 minutes | Check source workstation; confirm with user |
| 4625 across many accounts from one host | ≥ 3 accounts within 15 minutes | Treat as password spraying; isolate host |
| Any 4728/4756 for Domain Admins | Every occurrence | Verify against an approved change |
| Any 1102 | Every occurrence | Treat as possible tampering; compare with forwarded copy |
| 5136 on a GPO outside a change window | Every occurrence | Review the changed attribute and acting account |

---

## 4. Session Log

| Date | Test | Events observed | Notes |
| :--- | :--- | :--- | :--- |
| _pending_ | | | |
