# Audit Policy

The lab uses **Advanced Audit Policy Configuration** instead of the legacy nine
audit categories, because subcategories give more precise events with less noise.

**GPO path:** `Computer Configuration → Policies → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Audit Policies`

Required companion setting:
`Security Options → Audit: Force audit policy subcategory settings (Windows Vista or later) to override audit policy category settings` = **Enabled**

---

## 1. Subcategory Settings

| Category | Subcategory | DC01 | CLIENT01 | FS01 | Key events | Threat |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| Account Logon | Credential Validation | S/F | — | — | 4776 | T-01 |
| Account Logon | Kerberos Authentication Service | S/F | — | — | 4768, 4771 | T-01 |
| Account Management | User Account Management | S/F | S/F | — | 4720, 4722, 4724, 4738, 4740, 4767 | T-01, T-02 |
| Account Management | Security Group Management | S | S | S | 4728, 4732, 4756 | T-03, T-04 |
| Logon/Logoff | Logon | S/F | S/F | S/F | 4624, 4625 | T-01 |
| Logon/Logoff | Account Lockout | F | F | — | 4625 (lockout) | T-01 |
| Logon/Logoff | Special Logon | S | S | S | 4672 | T-04 |
| DS Access | Directory Service Changes | S | — | — | 5136, 5137, 5141 | T-04, T-06 |
| Policy Change | Audit Policy Change | S | S | S | 4719 | T-05 |
| Object Access | File Share | — | — | S/F | 5140 | T-07 |
| Object Access | Detailed File Share | — | — | F | 5145 | T-07 |
| Detailed Tracking | Process Creation | — | S | — | 4688 | Malware activity |
| System | Security State Change | S | S | S | 4608 | Startup baseline |

`S` = Success, `F` = Failure, `—` = not configured on that host.

The **Security log cleared** event (1102) is always generated and does not need a
subcategory.

---

## 2. Supporting Settings

| Setting | Value | Location |
| :--- | :--- | :--- |
| Include command line in process creation events | Enabled | Administrative Templates → System → Audit Process Creation |
| Security log maximum size | 262,144 KB (256 MB) | Event Log settings |
| Retention method | Overwrite events as needed | Event Log settings |
| SACL for Directory Service Changes | *Everyone* — Write all properties on `CN=Policies,CN=System` | ADSI Edit / AD Users and Computers (Advanced) |

> **Note:** Directory Service Changes (5136) only fires for objects that have an
> auditing entry (SACL). Without the SACL on the GPO container, GPO edits are not logged.

---

## 3. Verification Commands

```powershell
# Show the effective audit policy on a host
auditpol /get /category:*

# Confirm the GPOs applied to the computer
gpresult /r /scope:computer

# Force policy refresh after a change
gpupdate /force
```

Results of these checks are recorded in [test-results.md](test-results.md).
