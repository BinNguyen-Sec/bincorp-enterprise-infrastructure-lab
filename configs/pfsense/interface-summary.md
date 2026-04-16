# pfSense Interface Summary

## Interface Mapping
- WAN: VMware NAT network
- LAN / Server Zone: 10.10.10.1/24
- CLIENT Zone: 10.10.20.1/24
- VPN Zone: 10.10.50.0/24
- DMZ: 10.10.70.1/24

## Main Functions
- gateway and routing
- firewall enforcement
- client DHCP
- OpenVPN endpoint
- Suricata IDS
- remote syslog forwarding

## DHCP Scope
Client DHCP scope:
- network: 10.10.20.0/24
- range: 10.10.20.100 - 10.10.20.150
- DNS: 10.10.10.10, 10.10.10.11