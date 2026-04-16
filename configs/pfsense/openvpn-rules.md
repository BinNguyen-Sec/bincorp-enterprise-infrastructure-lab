# OpenVPN Rule Summary

## OpenVPN Network
- tunnel network: 10.10.50.0/24

## Allowed Access
VPN clients are allowed to:
- query DNS on DC01 (10.10.10.10:53)
- query DNS on DC02 (10.10.10.11:53)
- access HTTPS intranet on APP01 (10.10.10.20:443)

## Security Design
The permissive OpenVPN wizard rule was disabled and replaced with explicit least-privilege rules.

## Purpose
This configuration demonstrates secure remote access with restricted internal reachability rather than unrestricted VPN access.