# Hardening Checklist

Tick each item after it is applied **and** verified. Verification evidence (screenshots,
`gpresult` output) should be sanitized before publishing, as described in the
[asset inventory](../architecture/asset-inventory.md) classification table.

Status legend: `[ ]` not done · `[x]` applied and verified

---

## 1. Domain Controller (DC01)

- [ ] Static IP `10.10.10.10`, DNS points to itself
- [ ] Windows updates installed before promotion to domain controller
- [ ] Built-in `Administrator` renamed and given a long random password
- [ ] Separate `adm-` account created for daily administration
- [ ] Privileged groups (`Domain Admins`, `Enterprise Admins`, `Schema Admins`) reviewed
- [ ] OU structure created (`Lab-Workstations`, `Lab-Servers`, `Lab-Users`, `Lab-Admins`)
- [ ] Password and account lockout policy set in Default Domain Policy
- [ ] SEC-DC-Audit GPO linked and `auditpol` output verified
- [ ] SACL added to the GPO container for Directory Service Changes
- [ ] Security log size increased to 256 MB
- [ ] Windows Event Collector configured with a subscription for CLIENT01 and FS01
- [ ] Unused roles and features not installed

## 2. File Server (FS01)

- [ ] Static IP `10.10.10.20`, DNS `10.10.10.10`, joined to `corp.lab`
- [ ] Computer object moved to `Lab-Servers`
- [ ] One security group per departmental share (for example `GG-Share-Finance`)
- [ ] Share permissions: `Authenticated Users` → Change; NTFS permissions restrict access per group
- [ ] `Everyone` removed from share and NTFS ACLs
- [ ] SMBv1 disabled
- [ ] File Share auditing enabled and a SACL set on audited folders
- [ ] Event forwarding to DC01 verified

## 3. Workstation (CLIENT01)

- [ ] Joined to `corp.lab` and moved to `Lab-Workstations`
- [ ] Standard user is **not** a local administrator (Restricted Groups applied)
- [ ] Microsoft Defender Antivirus and real-time protection enabled
- [ ] Windows Defender Firewall enabled for Domain, Private and Public profiles
- [ ] SMBv1 disabled
- [ ] Guest account disabled
- [ ] Interactive logon message configured
- [ ] Screen lock after 15 minutes of inactivity
- [ ] Process creation auditing with command line enabled
- [ ] Event forwarding to DC01 verified

## 4. Hypervisor and Lab Hygiene

- [ ] All VMs use the `CORP-LAB` internal network only (no bridged adapter)
- [ ] Snapshot taken after each stable milestone
- [ ] No real personal or organizational credentials used in the lab
- [ ] Screenshots and exports sanitized before publishing (no passwords, hashes, tokens)

---

## Useful Verification Commands

```powershell
Get-SmbServerConfiguration | Select EnableSMB1Protocol
Get-LocalGroupMember -Group "Administrators"
Get-NetFirewallProfile | Select Name, Enabled
Get-ADGroupMember "Domain Admins"
net accounts /domain
```
