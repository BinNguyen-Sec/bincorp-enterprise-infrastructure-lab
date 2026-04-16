# Use Cases

## Overview
This section presents representative use cases used to validate the enterprise infrastructure lab for BinCorp. The scenarios focus on authentication monitoring, VPN access, proxy policy enforcement, and domain service redundancy. Each use case includes objective, components involved, execution steps, and expected results.

---

## Use Case 1: Failed Logon Detection

### Objective
Validate that failed domain authentication attempts are recorded by the Domain Controller and centralized into Splunk for monitoring and investigation.

### Components Involved
- CL01
- DC01 / DC02
- SPLUNK01
- Active Directory
- Splunk Universal Forwarder

### Preconditions
- CL01 is joined to `bincorp.local`
- DC01 and DC02 are operational
- Security Event Logs from DC01 and DC02 are forwarded to Splunk
- A domain user account exists, for example `reception.user`

### Execution Steps
1. On CL01, sign out from the current session.
2. At the Windows sign-in screen, enter a valid domain username such as `BINCORP\reception.user`.
3. Enter an incorrect password multiple times.
4. Observe that Windows denies the authentication attempt.
5. Return to SPLUNK01 and open Search & Reporting.
6. Run the following query:

```spl
index=main (host="DC01" OR host="DC02") source="WinEventLog:Security" EventCode=4625
```

### Expected Results
- Windows displays a failed logon message on CL01.
- A Security Event with EventCode=4625 appears in Splunk.
- The event contains indicators such as:
+ failed authentication
+ account name
+ logon category
+ audit failure

### Security Value

This use case demonstrates centralized visibility into failed authentication attempts, which supports security monitoring, account abuse detection, and audit review.

## Use Case 2: VPN Access Detection
### Objective

Validate that remote VPN access is functional and that VPN-related events from pfSense are collected into Splunk.

### Components Involved
- pfSense
- OpenVPN
- SPLUNK01
- APP01
- BinCorp Intranet

### Preconditions
- OpenVPN server is configured on pfSense
- A valid VPN user exists, for example vpnuser
- pfSense remote syslog is forwarding events to Splunk on UDP 5140
- Internal DNS and HTTPS intranet are operational

### Execution Steps
1. Launch OpenVPN Connect from the remote client.
2. Authenticate with the configured VPN account.
3. Confirm that the VPN tunnel is established and a VPN IP address is assigned.
4. After connection, browse to:
```bash
https://intranet.bincorp.local
```
5. On SPLUNK01, run the following query:
```spl
index=main host="10.10.10.1" "openvpn"
```

### Expected Results
- OpenVPN shows a successful connection state.
- The client receives an address from the VPN subnet, such as 10.10.50.2.
- The internal intranet is reachable through the VPN tunnel.
- Splunk shows OpenVPN events such as:
+ user authenticated
+ peer connection initiated
+ tunnel connected
+ IP assigned from VPN pool

### Security Value

This use case demonstrates secure remote access capability and centralized logging of VPN authentication and connection events.

## Use Case 3: Proxy Policy Enforcement
### Objective

Validate that web access policy is enforced through WinGate Proxy and that restricted content is blocked while allowed content remains accessible.

### Components Involved
- CL01
- APP01
- WinGate Proxy
- Web Access Control Rules

### Preconditions
- CL01 is configured to use proxy 10.10.10.20:8080
- WinGate is running on APP01
- A block rule for youtube.com is configured
- General outbound browsing through proxy is operational

### Execution Steps
1. From CL01, browse to an allowed website such as:
```bash
https://www.facebook.com
```
2. Confirm that the page loads successfully.
3. From the same client, browse to:
```bash
https://www.youtube.com
```
4. Confirm that the site does not load because of the proxy policy.
5.On APP01, open WinGate Management and check Activity / Access Rules.

### Expected Results
- Allowed websites remain accessible.
- Restricted websites are blocked by the proxy.
- WinGate shows web activity originating from CL01.
- The configured block policy is effectively enforced.

### Security Value

This use case demonstrates acceptable-use policy enforcement, controlled web access, and administrative restriction of selected web destinations.

## Use Case 4: Domain Controller Redundancy Validation
### Objective

Validate that domain services remain partially available when one Domain Controller is offline, demonstrating redundancy at the AD/DNS layer.

### Components Involved
- DC01
- DC02
- CL01
- APP01
- Active Directory
- DNS

### Preconditions
- DC01 and DC02 have been configured as replicated Domain Controllers
- AD replication was previously verified successfully
- Internal services are operational before the test begins

### Execution Steps
1. Confirm that the environment is operating normally.
2. Shut down or power off DC01.
3. From CL01, verify reachability to DC02:
ping 10.10.10.11
4. Validate that internal service continuity remains available, for example by opening:
```bash
https://intranet.bincorp.local
```
5. Optionally verify that DNS and AD tools remain available on DC02.

### Expected Results
- DC01 is offline.
- DC02 remains reachable.
- Internal services continue operating.
- Domain service does not completely collapse when one DC is lost.

### Notes

This lab validates redundancy at the Domain Controller layer, not full high availability for every application component. The application tier remains largely single-instance.

### Security / Availability Value

This use case demonstrates resilience in identity and directory services and supports business continuity at the domain service layer.

## Use Case 5: HTTPS Intranet with Internal PKI
### Objective

Validate that the internal web portal is protected with a certificate issued by the internal Enterprise CA.

### Components Involved
- DC01
- AD CS / BinCorp-RootCA
- APP01
- IIS
- CL01

### Preconditions
- Enterprise Root CA is deployed
- Web Server template is issued
- APP01 has a certificate for intranet.bincorp.local
- IIS HTTPS binding is configured

### Execution Steps
1. On CL01, browse to:
```bash
https://intranet.bincorp.local
```
2. Verify that the internal portal loads successfully over HTTPS.
3. Confirm certificate usage through IIS binding and local certificate store on APP01.

### Expected Results
- Intranet is accessible over HTTPS.
- A certificate issued by BinCorp-RootCA is bound to IIS.
- Internal PKI is actively used to secure the internal web application.

### Security Value

This use case demonstrates encryption in transit, trust management, and practical use of internal PKI in enterprise infrastructure.

# Summary

The implemented use cases demonstrate that the BinCorp enterprise infrastructure lab supports:

- centralized monitoring of failed logon attempts
- controlled VPN-based remote access
- policy-based web filtering through proxy
- redundancy at the domain service layer
- secure internal web access using enterprise PKI

These scenarios collectively show that the lab is not only a deployment exercise, but also a validation of operational security controls in a business-oriented environment.