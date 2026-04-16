# IP Plan

## Overview
This document defines the IP addressing plan for the BinCorp Enterprise Infrastructure Lab.  
The addressing model is designed to be simple, readable, and scalable enough for segmented enterprise-style lab deployment.

---

## Address Space
The lab uses the following private address range:

10.10.0.0/16

This allows subnet-based separation between server, client, VPN, and future expansion zones.

---

## Subnet Layout

| Zone | Purpose | Subnet | Gateway |
|---|---|---|---|
| Server Zone | Core servers | 10.10.10.0/24 | 10.10.10.1 |
| Client Zone | User endpoints | 10.10.20.0/24 | 10.10.20.1 |
| VPN Zone | OpenVPN clients | 10.10.50.0/24 | 10.10.50.1 |
| DMZ | Reserved future expansion | 10.10.70.0/24 | 10.10.70.1 |

---

## Core Host IP Assignments

| Host | Role | IP Address | Notes |
|---|---|---|---|
| pfSense (Server Gateway) | Gateway for Server Zone | 10.10.10.1 | Internal server gateway |
| pfSense (Client Gateway) | Gateway for Client Zone | 10.10.20.1 | Internal client gateway |
| pfSense (VPN Endpoint) | OpenVPN tunnel network | 10.10.50.1 | VPN subnet gateway |
| pfSense (DMZ Gateway) | Gateway for DMZ | 10.10.70.1 | Reserved for future expansion |
| DC01 | Primary Domain Controller | 10.10.10.10 | AD DS, DNS, AD CS |
| DC02 | Secondary Domain Controller | 10.10.10.11 | AD DS, DNS |
| APP01 | Internal application/service server | 10.10.10.20 | File Server, IIS, WinGate |
| SPLUNK01 | Monitoring server | 10.10.10.50 | Splunk Enterprise |
| CL01 | Domain client | DHCP in 10.10.20.0/24 | Example: 10.10.20.100 |

---

## Domain Information

| Item | Value |
|---|---|
| Domain Name | bincorp.local |
| NetBIOS Name | BINCORP |

---

## DNS Design

### Domain Controllers
- DC01 DNS: 10.10.10.10
- DC02 DNS: 10.10.10.11

### Recommended DNS Configuration

#### DC01
- Preferred DNS: 10.10.10.10
- Alternate DNS: 10.10.10.11

#### DC02
- Preferred DNS: 10.10.10.11
- Alternate DNS: 10.10.10.10

#### APP01
- Preferred DNS: 10.10.10.10
- Alternate DNS: 10.10.10.11

#### SPLUNK01
- Preferred DNS: 10.10.10.10
- Alternate DNS: 10.10.10.11

#### CL01
- Preferred DNS: 10.10.10.10
- Alternate DNS: 10.10.10.11

---

## DHCP Design

### DHCP Provider
pfSense currently provides DHCP for the Client Zone.

### Client DHCP Scope

| Setting | Value |
|---|---|
| Network | 10.10.20.0/24 |
| DHCP Range | 10.10.20.100 - 10.10.20.150 |
| Gateway | 10.10.20.1 |
| DNS 1 | 10.10.10.10 |
| DNS 2 | 10.10.10.11 |

---

## VPN Addressing

### OpenVPN Tunnel Network
10.10.50.0/24

### Example Assignment
- VPN client: 10.10.50.2

The VPN subnet is routed through pfSense and restricted by explicit OpenVPN firewall rules.

---

## Internal DNS Records

| Record | FQDN | IP Address |
|---|---|---|
| dc01 | dc01.bincorp.local | 10.10.10.10 |
| dc02 | dc02.bincorp.local | 10.10.10.11 |
| app01 | app01.bincorp.local | 10.10.10.20 |
| intranet | intranet.bincorp.local | 10.10.10.20 |
| splunk01 | splunk01.bincorp.local | 10.10.10.50 |

---

## Summary
The current IP plan supports:
- segmented network structure
- static addressing for servers
- DHCP-based addressing for clients
- dual DNS servers for the domain
- dedicated VPN subnet
- reserved DMZ subnet for future use

This structure is sufficient for the current BinCorp lab and can be expanded later if additional hosts or services are added.