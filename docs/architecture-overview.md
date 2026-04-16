# Architecture Overview

## Project Context
The BinCorp Enterprise Infrastructure Lab simulates the IT environment of a building management and office leasing company. The architecture combines identity services, internal business services, remote access, web filtering, intrusion detection, and centralized monitoring in one virtualized environment.

The design goal is to reflect an enterprise-style infrastructure while remaining practical to deploy in a personal lab.

---

## Core Design Principles
The architecture was built around the following principles:

- centralized identity and authentication
- separation of network roles by zone
- practical role-based access control
- secure internal services using internal PKI
- controlled remote access
- centralized monitoring and log visibility
- partial resilience at the domain service layer

---

## Main Systems

### pfSense
pfSense acts as the network edge and security gateway for the environment.

Main functions:
- WAN connectivity
- internal routing
- firewall rule enforcement
- subnet separation
- OpenVPN remote access
- Suricata IDS deployment
- syslog forwarding to Splunk

---

### DC01
DC01 is the primary Domain Controller.

Main functions:
- Active Directory Domain Services
- DNS
- identity and authentication services
- internal PKI role through AD CS

---

### DC02
DC02 is the secondary Domain Controller.

Main functions:
- redundant AD DS
- redundant DNS
- replication partner for DC01
- continuity at the domain service layer

---

### APP01
APP01 is the internal service server.

Main functions:
- File Server
- IIS intranet hosting
- HTTPS internal portal
- WinGate Proxy

This server concentrates multiple business services into one node for lab practicality.

---

### SPLUNK01
SPLUNK01 is the centralized monitoring platform.

Main functions:
- Splunk Enterprise
- local event ingestion
- receiving Windows logs via Universal Forwarder
- receiving pfSense syslog
- search and monitoring use cases

---

### CL01
CL01 is the domain-joined endpoint used for validation and testing.

Main functions:
- domain logon validation
- file share access testing
- proxy usage testing
- intranet access testing
- failed logon testing

---

## Logical Network Layout

### WAN
External connectivity is provided through the VMware NAT-backed WAN interface on pfSense.

### Server Zone
This zone hosts the main server infrastructure:
- DC01
- DC02
- APP01
- SPLUNK01

### Client Zone
This zone is used for user endpoint testing:
- CL01

### DMZ
A DMZ segment is defined in the design for future expansion.

### VPN Zone
OpenVPN clients are assigned addresses from a dedicated VPN subnet.

---

## Security Layers in the Design

### Identity Layer
- Active Directory
- security groups
- domain-based access control

### Network Layer
- pfSense firewall
- segmentation between internal zones
- OpenVPN access rules

### Service Layer
- internal file sharing
- intranet web portal
- proxy-based web usage control

### Trust Layer
- AD CS
- internal certificates
- HTTPS intranet

### Detection and Monitoring Layer
- Suricata IDS
- Splunk Enterprise
- Windows Event Log forwarding
- pfSense syslog forwarding

---

## Architectural Strengths
The architecture provides:

- centralized identity control
- role-based access management
- internal certificate-based trust
- secure remote access
- proxy-enforced browsing policy
- centralized event visibility
- redundancy for AD/DNS through dual Domain Controllers

---

## Architectural Limitations
The current architecture still has practical lab limitations:

- APP01 is multi-role and remains a single-instance dependency
- pfSense is a single firewall instance
- Splunk is a single monitoring instance
- application-layer high availability is not fully implemented
- file services are not yet DFS-based

These limitations are acceptable for a lab environment and are documented rather than hidden.

---

## Conclusion
The BinCorp lab architecture is designed to reflect how multiple enterprise technologies operate together rather than as isolated tools. The result is a layered infrastructure model that supports identity, secure internal services, remote access, detection, and monitoring in a realistic virtual lab.