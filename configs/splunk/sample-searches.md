# Sample Searches

## Failed Logon Detection
```spl
index=main (host="DC01" OR host="DC02") source="WinEventLog:Security" EventCode=4625
``` 

DC01 Events
```spl
index=main host="DC01"
```

DC02 Events
```spl
index=main host="DC02"
```

APP01 Events
```spl
index=main host="APP01"
```

VPN Events from pfSense
```spl
index=main host="10.10.10.1" "openvpn"
```

General pfSense Syslog
```spl
index=main source=*syslog*
```

Security Event Logs
```spl
index=main source="WinEventLog:Security"
```