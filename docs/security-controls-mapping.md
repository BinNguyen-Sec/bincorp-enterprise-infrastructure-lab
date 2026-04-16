# Security Controls Mapping

## Overview
This document maps the implemented components of the BinCorp enterprise infrastructure lab to practical security control areas. The purpose is not to claim full compliance with any formal certification program, but to demonstrate how the deployed technical controls align with enterprise security design principles and with the general direction of ISO/IEC 27001-style governance.

The environment was designed to combine identity services, access control, secure remote access, internal PKI, monitoring, web filtering, and centralized log analysis into a cohesive enterprise lab.

---

## 1. Identity and Access Control

### Implemented Components
- Active Directory Domain Services on `DC01` and `DC02`
- Domain-joined servers and client:
  - `APP01`
  - `CL01`
  - `SPLUNK01`
- Security groups for department-based access
- NTFS and share permissions on file shares
- Controlled access to privileged folders such as `IT`

### Technical Controls
- Domain-based authentication for users and systems
- Role-based access through security groups
- Department-level segregation of file access
- Restricted access to high-privilege shared resources
- Domain account monitoring through Security Event Logs

### Security Purpose
These controls reduce anonymous access, centralize identity management, and enforce role-based restrictions across users and shared resources.

### Relevant Control Theme
- Identity management
- Authentication
- Access control
- Least privilege

---

## 2. Redundancy and Availability of Domain Services

### Implemented Components
- `DC01` as primary Domain Controller
- `DC02` as additional Domain Controller
- Active Directory replication
- DNS service on both domain controllers

### Technical Controls
- Replication validated with `repadmin /replsummary`
- DNS zone present across both DCs
- Domain services remain available when one Domain Controller is offline

### Security Purpose
These controls improve resilience of authentication and directory services and reduce the likelihood of a single point of failure at the identity layer.

### Relevant Control Theme
- Availability
- Business continuity
- Resilience of critical infrastructure

---

## 3. Network Segmentation and Firewall Control

### Implemented Components
- pfSense as edge firewall and gateway
- Separate logical zones:
  - WAN
  - LAN / Server Zone
  - CLIENT Zone
  - DMZ
  - VPN subnet

### Technical Controls
- Interface-based separation of zones
- Firewall rules controlling movement between networks
- Dedicated CLIENT subnet with DHCP
- OpenVPN subnet with explicit least-privilege rules
- Restricted VPN access to selected internal services

### Security Purpose
These controls reduce unnecessary lateral movement and enforce traffic boundaries between network zones.

### Relevant Control Theme
- Network security
- Segmentation
- Perimeter control
- Traffic restriction

---

## 4. Secure Remote Access

### Implemented Components
- OpenVPN on pfSense
- Local VPN user and certificate-based VPN setup
- VPN subnet `10.10.50.0/24`
- VPN-specific firewall policy

### Technical Controls
- OpenVPN server certificate
- VPN client certificate
- User authentication for tunnel access
- Explicit rules allowing VPN clients only to:
  - internal DNS
  - intranet HTTPS

### Security Purpose
These controls support secure remote access while limiting what a connected VPN user can reach.

### Relevant Control Theme
- Remote access security
- Secure communications
- Least privilege
- Boundary enforcement

---

## 5. Internal PKI and Encryption

### Implemented Components
- Active Directory Certificate Services
- `BinCorp-RootCA`
- Web Server certificate for `intranet.bincorp.local`
- HTTPS binding on IIS

### Technical Controls
- Enterprise Root CA deployment
- Certificate issuance for internal web service
- Internal trust chain
- HTTPS on intranet site

### Security Purpose
These controls provide encryption in transit and establish trust for internal services through enterprise certificate management.

### Relevant Control Theme
- Cryptographic controls
- Certificate lifecycle
- Secure internal services
- Trust establishment

---

## 6. File Access Protection

### Implemented Components
- File Server role on `APP01`
- Shared folders for departments
- Public and restricted file shares
- Group-based share and NTFS permissions

### Technical Controls
- File access granted per department group
- Restricted access to sensitive folders
- Differentiation between public and private resources
- Administrative full control retained for management purposes

### Security Purpose
These controls reduce unauthorized file access and implement practical role-based file security.

### Relevant Control Theme
- Information access restriction
- Data segregation
- Resource authorization

---

## 7. Proxy Enforcement and Web Usage Control

### Implemented Components
- WinGate Proxy on `APP01`
- Proxy settings on `CL01`
- Web Access Control rules
- Explicit block policy for selected websites

### Technical Controls
- Proxy-mediated outbound browsing
- Site-specific deny rule for `youtube.com`
- Activity monitoring of proxied traffic
- Validation that allowed and denied destinations behave differently

### Security Purpose
These controls enforce acceptable use restrictions and create administrative control over web access.

### Relevant Control Theme
- Acceptable use
- Web filtering
- User activity control
- Policy enforcement

---

## 8. Intrusion Detection

### Implemented Components
- Suricata on pfSense
- Rule updates from ET Open and community sources
- Suricata interfaces on WAN and CLIENT
- Alert logging enabled

### Technical Controls
- IDS deployment at the network layer
- Rule-based inspection of network traffic
- Alert generation validated in the lab environment
- WAN and CLIENT inspection coverage

### Security Purpose
These controls support network monitoring and detection of suspicious or abnormal events.

### Relevant Control Theme
- Threat detection
- Network monitoring
- Event visibility

### Note
The Suricata implementation in this lab demonstrates IDS functionality and event generation in a virtualized environment. It is intended as a lab-grade detection layer rather than a tuned production IDS deployment.

---

## 9. Logging and Centralized Monitoring

### Implemented Components
- Splunk Enterprise on `SPLUNK01`
- Splunk Universal Forwarder on:
  - `DC01`
  - `DC02`
  - `APP01`
- Windows Event Log ingestion
- pfSense syslog ingestion into Splunk

### Technical Controls
- Centralized log aggregation in Splunk
- Windows Security/System/Application event forwarding
- pfSense syslog forwarding over UDP
- Searchable security use cases in Splunk

### Security Purpose
These controls provide central visibility across hosts and network devices, enabling investigation and operational monitoring.

### Relevant Control Theme
- Logging
- Event monitoring
- Audit trail
- Security visibility

---

## 10. Security Monitoring Use Cases

### Implemented Use Cases
- Failed domain logon detection (`EventCode=4625`)
- VPN connection detection through pfSense syslog
- Proxy block validation
- Domain controller redundancy validation
- HTTPS intranet validation with internal PKI

### Technical Controls
- Use-case-driven testing against real generated events
- Correlation of host and firewall data in Splunk
- Visual validation of access control and service continuity

### Security Purpose
These use cases confirm that the implemented controls are not merely installed, but operational and testable.

### Relevant Control Theme
- Security monitoring
- Validation
- Operational assurance

---

## 11. Backup and Recovery Considerations

### Implemented / Documented Considerations
- pfSense configuration can be backed up
- Windows VMs can be protected by VM-level backup/snapshot strategy
- Dual DC design reduces single-point failure at domain layer
- Application tier remains mostly single-instance in this lab

### Security Purpose
These considerations support recovery planning and improve resilience of core infrastructure.

### Relevant Control Theme
- Backup
- Recovery
- Continuity planning

### Note
This lab includes recovery-oriented design considerations, but it does not implement a full enterprise backup platform.

---

## 12. Summary Mapping

| Security Area | Implemented Component(s) | Practical Outcome |
|---|---|---|
| Identity and authentication | AD DS, domain join, security groups | Centralized authentication and role-based access |
| Directory resilience | DC01, DC02, replication, DNS | Redundant domain services |
| Network protection | pfSense, segmented zones, firewall rules | Controlled communication paths |
| Remote access security | OpenVPN, VPN rules, certificates | Restricted remote connectivity |
| Cryptography | AD CS, HTTPS intranet | Trusted internal encryption |
| File protection | File shares, NTFS permissions, group access | Department-based data access |
| Web control | WinGate Proxy, block rules | Enforced browsing policy |
| Intrusion detection | Suricata | Network event detection |
| Monitoring and audit | Splunk, forwarders, syslog | Centralized log visibility |
| Continuity | Dual DC, documented backup/DR notes | Improved resilience at core service layer |

---

## Conclusion
The BinCorp enterprise infrastructure lab demonstrates a layered security design rather than a single isolated technology stack. Identity, network control, remote access, encryption, proxy enforcement, IDS, and SIEM were implemented as interacting controls.

The result is a practical enterprise-style lab that supports:
- centralized authentication
- segmented network access
- secure internal services
- controlled remote access
- basic threat detection
- centralized event analysis
- resilience at the domain service layer

While not a full production-grade environment, the lab meaningfully reflects enterprise security architecture and provides a strong technical foundation for documentation, portfolio presentation, and CV discussion.