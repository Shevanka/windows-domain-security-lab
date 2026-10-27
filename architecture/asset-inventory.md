# Asset Inventory

## Environment Scope

This project is an isolated Windows domain home lab built in Oracle
VirtualBox. It simulates centralized identity, endpoint administration,
security policy enforcement, and event monitoring in a small enterprise
environment.

## Systems

### DC01

- Role: Domain controller
- Operating system: Windows Server
- Private address: 10.10.10.10
- Services:
  - Active Directory Domain Services
  - Domain Name System
  - Group Policy
  - Windows Event Collection, if applicable

### CLIENT01

- Role: Domain workstation
- Operating system: Windows 11
- Address allocation: DHCP or static lab address
- Domain membership: corp.lab
- Primary user: Authorized test account

## Security Boundaries

- VirtualBox host
- CORP-LAB internal network
- Active Directory domain
- Domain controller
- Domain workstation
- Administrative accounts

## Exclusions

- Production systems
- Public-facing services
- Real employee information
- Real organizational credentials
- Third-party cloud infrastructure

## Ringkasan Matriks Klasifikasi

| Classification    | Examples                                | Handling                   |
| ----------------- | --------------------------------------- | -------------------------- |
| Public            | Architecture overview, generic policies | May be published           |
| Internal lab      | Hostnames, test-user structure          | Sanitize before publishing |
| Sensitive         | Passwords, keys, internal addresses     | Never publish              |
| Security evidence | Event logs and GPO exports              | Review and sanitize        |
