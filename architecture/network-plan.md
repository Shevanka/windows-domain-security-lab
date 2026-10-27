# Network Plan

## Virtual Network

- Network name: CORP-LAB
- Network type: VirtualBox Internal Network
- Subnet: 10.10.10.0/24
- Subnet mask: 255.255.255.0

## Address Plan

- DC01: 10.10.10.10
- FS01: 10.10.10.20
- CLIENT01: 10.10.10.100
- DHCP pool: 10.10.10.100–10.10.10.199
- DNS server: 10.10.10.10

## Trust Assumptions

- The VirtualBox host is trusted.
- Domain administrator credentials are used only for administration.
- Standard users do not receive local administrator rights.
- Only approved machines may join the domain.

## Network Limitations

The first iteration does not include network segmentation, a dedicated
firewall, an IDS, redundant domain controllers, or secure remote access.
