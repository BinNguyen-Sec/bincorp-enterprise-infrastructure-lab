# Deployment Guide

## Overview
This document summarizes the deployment order used to build the BinCorp Enterprise Infrastructure Lab. The sequence was intentionally designed to reduce dependency issues and keep troubleshooting manageable.

The strategy was:
1. build the network foundation
2. deploy core identity services
3. deploy internal application services
4. add trust and secure remote access
5. add detection and monitoring
6. validate the environment through use cases

---

## Phase 1 — VMware Network and pfSense

### Objective
Build the network foundation of the lab.

### Main Tasks
- create the VMware virtual networks
- define WAN, Server, Client, and DMZ connectivity
- deploy pfSense
- assign and configure interfaces
- configure DHCP for the Client Zone
- configure basic firewall rules
- configure OpenVPN rules later in the process

### Result
The lab gained a working routed foundation with internal subnets and a controllable security gateway.

---

## Phase 2 — Primary Domain Controller (DC01)

### Objective
Deploy the first identity and DNS node.

### Main Tasks
- deploy DC01
- assign static IP `10.10.10.10`
- join Server Zone
- install AD DS and DNS
- create the domain:
  - `bincorp.local`

### Result
The environment gained centralized identity and DNS services.

---

## Phase 3 — Secondary Domain Controller (DC02)

### Objective
Introduce redundancy at the AD/DNS layer.

### Main Tasks
- deploy DC02
- assign static IP `10.10.10.11`
- join domain
- install AD DS and DNS
- promote as additional Domain Controller
- verify replication
- validate Sites and Services
- verify `repadmin /replsummary`

### Result
The environment gained domain-layer redundancy and replicated DNS.

---

## Phase 4 — Client Validation (CL01)

### Objective
Validate that a user endpoint can function in the domain.

### Main Tasks
- deploy CL01 in Client Zone
- receive DHCP from pfSense
- use domain DNS servers
- join `bincorp.local`
- validate domain authentication

### Result
The lab gained a working domain client for user-level testing.

---

## Phase 5 — APP01 and Internal Business Services

### Objective
Deploy the application/service node.

### Main Tasks
- deploy APP01
- assign static IP `10.10.10.20`
- join the domain
- install File Server role
- create shared folders
- implement security groups and permissions
- deploy IIS
- publish intranet page

### Result
The environment gained internal business services:
- file sharing
- intranet web service

---

## Phase 6 — PKI and HTTPS

### Objective
Secure the internal intranet with internal PKI.

### Main Tasks
- install AD CS on DC01
- configure Enterprise Root CA:
  - `BinCorp-RootCA`
- issue Web Server certificate template
- request certificate for `intranet.bincorp.local`
- bind certificate to IIS HTTPS
- validate HTTPS access from CL01

### Result
The intranet became accessible over HTTPS using an internally trusted certificate.

---

## Phase 7 — Proxy and Remote Access

### Objective
Add controlled web access and secure remote access.

### Main Tasks — Proxy
- install WinGate on APP01
- configure CL01 to use proxy
- validate normal web access
- create site block rule
- validate restricted site behavior

### Main Tasks — VPN
- configure OpenVPN on pfSense
- create VPN CA and certificates
- export VPN client profile
- validate VPN connection
- restrict VPN access with explicit firewall rules

### Result
The environment gained:
- proxy-based web filtering
- secure remote access with least-privilege behavior

---

## Phase 8 — IDS Deployment

### Objective
Add a network-based detection layer.

### Main Tasks
- install Suricata on pfSense
- update rule sources
- deploy Suricata to WAN and CLIENT
- run in IDS mode
- validate alert generation

### Result
The environment gained IDS visibility and basic event detection at the network layer.

---

## Phase 9 — Centralized Monitoring with Splunk

### Objective
Centralize host and firewall logs.

### Main Tasks
- deploy SPLUNK01
- assign static IP `10.10.10.50`
- join domain
- install Splunk Enterprise
- ingest local Windows event logs
- configure receiving port `9997`
- install Universal Forwarder on:
  - DC01
  - DC02
  - APP01
- configure `inputs.conf` and `outputs.conf`
- receive pfSense syslog over UDP `5140`

### Result
The environment gained a functional SIEM-style monitoring layer.

---

## Phase 10 — Use Case Validation

### Objective
Validate that the deployed controls actually function.

### Validated Use Cases
- failed logon detection in Splunk
- VPN access detection in Splunk
- proxy block enforcement
- domain controller redundancy validation
- HTTPS intranet validation

### Result
The project moved from installation-only status to demonstrated operational validation.

---

## Final State
At the end of deployment, the lab includes:

- segmented network edge with pfSense
- dual Domain Controllers with replication
- internal file and web services
- internal PKI and HTTPS
- controlled proxy-based web access
- OpenVPN remote access
- Suricata IDS
- Splunk-based centralized monitoring

---

## Deployment Order Summary

| Phase | Main Outcome |
|---|---|
| Phase 1 | Network foundation with pfSense |
| Phase 2 | Primary AD/DNS services |
| Phase 3 | Redundant AD/DNS services |
| Phase 4 | Domain client validation |
| Phase 5 | File Server and intranet services |
| Phase 6 | Internal PKI and HTTPS |
| Phase 7 | Proxy and VPN |
| Phase 8 | IDS deployment |
| Phase 9 | Centralized monitoring with Splunk |
| Phase 10 | Use case validation |

---

## Conclusion
The deployment process was intentionally layered so that each service was built on top of a verified dependency chain. This reduced unnecessary troubleshooting and made the final environment easier to validate, document, and present.