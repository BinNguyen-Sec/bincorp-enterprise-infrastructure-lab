# Splunk Receiving Summary

## SPLUNK01
- host: 10.10.10.50
- web interface: http://10.10.10.50:8000

## Receiving Ports
- TCP 9997: Universal Forwarder input
- UDP 5140: pfSense syslog input

## Data Sources
- local Windows event logs on SPLUNK01
- DC01 via Universal Forwarder
- DC02 via Universal Forwarder
- APP01 via Universal Forwarder
- pfSense syslog via UDP input
