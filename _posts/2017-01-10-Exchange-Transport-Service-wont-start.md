---
layout: post
title: Exchange Transport Service wont start
---
### Exchange Transport Service wont start.

One day I came back and saw that all my exchange services are disabled.

THis post helped me. http://www.itguydiaries.net/2012/08/omg-all-exchange-services-are-disabled.html

Specifically ran these two in powershell
```
Get-Service | Where-Object { $_.DisplayName –ilike “Microsoft Exchange *” } | Set-Service –StartupType Automatic
Get-Service | Where-Object { $_.DisplayName –ilike “Microsoft Exchange *” } | Start-Service
```


This started all the xchg services except transport service. That kept saying: Dependency service or group failed to start.

Checked if ipv6 was enabled? It was not. Enabled it and trying again.

Login to exchange 2013 admin console from [here] (https://localhost/ecp/?ExchClientVer=15)

# Solution:
Finally found out that, on services property in dependecy tab you can view which services are required. I found one of the dependency was down. 
Once I made that up, transport service came up alright.
