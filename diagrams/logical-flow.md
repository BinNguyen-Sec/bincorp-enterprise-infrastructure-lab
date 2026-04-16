# Logical Flow

## Network Flow
Internet traffic enters the lab through pfSense, which acts as the firewall, router, VPN endpoint, and segmentation control point.

### Internal Segments
- Server Zone: hosts core infrastructure servers
- Client Zone: hosts user endpoint simulation
- DMZ: reserved for future expansion
- VPN Zone: assigned to OpenVPN clients

## Identity Flow
CL01 and internal servers authenticate against the Active Directory domain `bincorp.local` through DC01 and DC02.

## DNS Flow
Internal name resolution is provided by the domain controllers:
- DC01: 10.10.10.10
- DC02: 10.10.10.11

## File Access Flow
Users authenticate to the domain, then access shared folders on APP01 based on group membership and NTFS/share permissions.

## Intranet Flow
Users browse to `https://intranet.bincorp.local`, which resolves to APP01 and is secured using a certificate issued by BinCorp-RootCA.

## Proxy Flow
CL01 routes web access through WinGate Proxy on APP01. Proxy policy allows or blocks traffic based on configured access rules.

## VPN Flow
Remote users connect to pfSense through OpenVPN and receive addresses from the VPN subnet `10.10.50.0/24`. Access is restricted by OpenVPN firewall rules.

## Monitoring Flow
Windows servers forward event logs to SPLUNK01 using Splunk Universal Forwarder. pfSense sends syslog to SPLUNK01 over UDP 5140.