# Backup and DR Note

## Overview
This document summarizes backup priorities, recovery considerations, and disaster recovery observations for the BinCorp enterprise infrastructure lab. The purpose is to define a realistic recovery mindset for the implemented environment, even though this lab does not include a full enterprise backup platform.

The backup and DR approach focuses on protecting the most critical services first, minimizing recovery time for core infrastructure, and identifying where the current lab still has single points of failure.

---

## 1. Recovery Priorities

The environment does not assign equal importance to all systems. Recovery should follow the service dependency chain.

### Priority 1 — Network Edge / Firewall
**System:** pfSense

#### Reason
pfSense is the entry point for:
- internal routing
- outbound connectivity
- VPN access
- firewall enforcement
- segmentation between zones

If pfSense is unavailable, most of the lab loses network functionality regardless of the health of the remaining servers.

#### Recovery Objective
Restore pfSense first so that internal and remote network communication can resume.

---

### Priority 2 — Domain Services
**Systems:** DC01, DC02

#### Reason
The domain controllers provide:
- Active Directory authentication
- DNS resolution
- group-based identity management
- trust relationships for domain-joined systems

These services are foundational for logon, name resolution, and system trust.

#### Recovery Objective
Ensure at least one functioning Domain Controller is available as quickly as possible.

---

### Priority 3 — Application and Internal Service Layer
**System:** APP01

#### Reason
APP01 hosts multiple functions:
- file shares
- intranet web service
- HTTPS internal portal
- WinGate Proxy

This makes it an important service aggregation point.

#### Recovery Objective
Restore file, proxy, and intranet services after the firewall and domain layer are stable.

---

### Priority 4 — Monitoring and Visibility
**System:** SPLUNK01

#### Reason
Splunk provides:
- centralized visibility
- event search
- forensic review capability
- monitoring of host and firewall events

Loss of Splunk reduces monitoring capability, but does not immediately stop core infrastructure operation.

#### Recovery Objective
Restore centralized monitoring after the operational infrastructure is functioning again.

---

### Priority 5 — Client Layer
**System:** CL01

#### Reason
CL01 is primarily a validation and user-access client. It is not a service provider.

#### Recovery Objective
Restore as needed for testing, operations validation, or user simulation.

---

## 2. Backup Scope

### 2.1 pfSense
#### Recommended Backup Items
- pfSense configuration XML
- interface assignments
- firewall rules
- OpenVPN configuration
- certificates and certificate authorities
- Suricata configuration summary
- remote logging settings

#### Suggested Backup Method
- export pfSense configuration from the web interface
- keep versioned copies of configuration exports
- optionally keep a VM snapshot before major changes

#### Why It Matters
pfSense is a configuration-heavy platform. Rebuilding manually is possible, but slower and more error-prone than restoring a known-good config.

---

### 2.2 Domain Controllers
#### Recommended Backup Items
- VM-level backup or snapshot strategy
- system state backup if using native Windows backup methods
- documentation of:
  - server name
  - IP
  - DNS configuration
  - domain role

#### Suggested Backup Method
- VM snapshots before major changes
- VM backups for recovery testing
- avoid relying only on snapshots for long-term protection

#### Why It Matters
Domain Controllers are critical to authentication and DNS. In this lab, dual DC design already improves resilience, but backup still matters in case both DCs fail or data corruption occurs.

---

### 2.3 APP01
#### Recommended Backup Items
- shared folder contents
- NTFS/share permission design notes
- IIS configuration and intranet content
- certificate bindings
- WinGate configuration and policy settings

#### Suggested Backup Method
- VM-level backup
- copy backup of shared data
- export or document IIS and proxy settings
- keep a copy of the intranet HTML content

#### Why It Matters
APP01 is a consolidated application/service node in this lab. A failure here affects:
- file sharing
- web intranet
- web proxy access

---

### 2.4 SPLUNK01
#### Recommended Backup Items
- Splunk local configuration
- data input settings
- receiving port configuration
- search queries and dashboards if created
- Universal Forwarder configuration files collected in the repo

#### Suggested Backup Method
- VM-level backup
- backup of Splunk configuration directories
- save useful search queries in documentation

#### Why It Matters
Losing Splunk does not stop infrastructure operation, but it removes centralized monitoring and investigation capability.

---

### 2.5 Certificates and PKI
#### Recommended Backup Items
- CA configuration notes
- CA name and role
- issued certificate inventory
- IIS-bound certificate details
- certificate template references

#### Suggested Backup Method
- VM-level protection of the CA host
- documentation of certificate names, purposes, and bindings
- export certificates where appropriate

#### Why It Matters
Internal PKI becomes operationally important once HTTPS and service trust depend on it.

---

## 3. Recovery Scenarios

### Scenario A — pfSense Failure
#### Impact
- internal routing disrupted
- VPN unavailable
- segmentation lost
- outbound connectivity lost
- syslog forwarding interrupted

#### Recovery Action
1. restore pfSense VM
2. restore pfSense configuration
3. verify interfaces, routing, firewall, VPN, and syslog settings
4. validate internal and remote access

#### Operational Note
This is the highest-priority single restoration point in the environment.

---

### Scenario B — Single Domain Controller Failure
#### Impact
- reduced redundancy
- one DNS/AD node unavailable
- authentication and DNS may still continue through remaining DC

#### Recovery Action
1. verify the remaining DC is healthy
2. restore failed DC when practical
3. confirm DNS and AD replication after recovery

#### Operational Note
This scenario was partially validated in the lab. The environment tolerates loss of one DC, though client-side DNS behavior may not switch instantly.

---

### Scenario C — APP01 Failure
#### Impact
- file shares unavailable
- intranet unavailable
- proxy unavailable
- HTTPS internal portal unavailable

#### Recovery Action
1. restore APP01 VM
2. verify DNS resolution to APP01
3. restore file share access
4. verify IIS and certificate binding
5. verify WinGate service and proxy rules

#### Operational Note
APP01 remains a major single-instance dependency in this lab.

---

### Scenario D — SPLUNK01 Failure
#### Impact
- no centralized visibility
- no search across Windows and pfSense logs
- monitoring and investigation degraded

#### Recovery Action
1. restore SPLUNK01 VM
2. confirm web access on port 8000
3. confirm receiving on port 9997 and UDP 5140
4. verify forwarder connectivity from DC01/DC02/APP01
5. verify pfSense syslog intake

#### Operational Note
Monitoring is degraded, but business services can still operate while Splunk is offline.

---

## 4. Practical DR Limitations in This Lab

### 4.1 Dual DC redundancy exists, but application HA is limited
The lab includes redundancy for:
- Active Directory
- DNS

The lab does **not** fully implement high availability for:
- file services
- IIS intranet
- proxy services
- Splunk
- pfSense

### 4.2 APP01 is a concentration point
APP01 currently combines:
- file services
- intranet web service
- proxy service

This is practical for a lab, but not ideal for production-style service separation.

### 4.3 DNS failover behavior on clients is not instantaneous
Even with dual DC/DNS, Windows DNS client behavior may not immediately switch to the secondary server during short failover tests. This affects user-facing name resolution demos, but does not invalidate the presence of domain service redundancy.

### 4.4 No dedicated enterprise backup platform is implemented
The lab relies conceptually on:
- VM snapshots
- VM-level backups
- configuration exports
- documentation of settings

This is acceptable for a technical lab, but not equivalent to a full enterprise backup strategy.

---

## 5. Improvement Opportunities

If the lab were expanded further, the following improvements would strengthen backup and DR maturity:

### Recommended Future Enhancements
- add a second application/file node
- implement DFS for file service redundancy
- separate proxy and intranet roles from file services
- protect Splunk with more formal configuration backup
- implement scheduled configuration exports for pfSense
- document a full restore checklist for each major server
- introduce backup validation as a test scenario

---

## 6. Recovery Checklist Summary

| Component | Priority | Main Dependency | Recovery Focus |
|---|---|---|---|
| pfSense | Highest | All internal and external traffic | Restore routing, firewall, VPN |
| DC01 / DC02 | High | Authentication and DNS | Restore at least one healthy DC |
| APP01 | High | File, intranet, proxy | Restore service node functions |
| SPLUNK01 | Medium | Monitoring and visibility | Restore logging/search capability |
| CL01 | Low | User validation/testing | Restore only as needed |

---

## 7. Conclusion
The BinCorp enterprise infrastructure lab includes meaningful resilience at the identity layer through dual Domain Controllers, but it still contains single-instance dependencies at the application and monitoring layers.

A realistic DR interpretation of the current lab is:
- **core identity services are partially resilient**
- **network edge remains critical**
- **application services are available but not fully HA**
- **monitoring is centralized but single-instance**

This balance is appropriate for an enterprise infrastructure lab intended for learning, documentation, portfolio presentation, and technical discussion.