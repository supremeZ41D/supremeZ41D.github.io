---
layout: default
title: "Netskope Traffic anaysis"
permalink: /netskope/nsanalysis/
---

# Agent Disabled - Packet Flow
This is an attempt to analyze how the Netskope agent behaves when disabled, when the computer has no other applications opened. I ran a Wireshark capture and the following findings are what was seen in it.

**Requisites**:
- The agent is already installed.
- The agent is user disabled.
- **Elapsed Time**: 12:47 min
- **Time Span**: 767.367s
- **Size**: 194MB
- **Interface**: Wifi
- **Total Packets**: 135928

**Netskope Findings**:
These are the Netskope domains found in the Wireshark capture:

| Domain                           | CNAME                                   | IP              | SNI                              |
| -------------------------------- | --------------------------------------- | --------------- | -------------------------------- |
| events.goskope.com               |                                         | 163.116.131.96  | events.goskope.com               |
| addon-<tenantname>.goskope.com    | ingress-lbaas-addonman.<mgmt_plane>.goskope.com | 163.116.131.73  | addon-<tenantname>.goskope.com    |
| gateway.gslb.goskope.com         |                                         | 163.116.148.27  | gateway.gslb.goskope.com         |
| achecker-<tenantname>.goskope.com | achecker.<mgmt_plane>.goskope.com               | 163.116.131.156 | achecker-<tenantname>.goskope.com |

When performing a _dig_ request on those domains, and subsequently on the _opskope_ domains, the following NS servers are seen:

| goskope NS Servers      | opskope NS server  |
| ----------------------- | ------------------ |
| ns1.goskope.opskope.com | ns1glr.goskope.com |
| ns2.goskope.opskope.com | ns2cvx.goskope.com |
| ns3.goskope.com         | ns3ghw.goskope.com |
| ns4.goskope.com         | ns4lny.goskope.com |
| ns5.goskope.com         |                    |
| ns6.goskope.com         |                    |

For more context about those domains (please refer to https://docs.netskope.com):
- **_events.goskope.com_** — It collects telemetry/event data for monitoring user experience.
- **_addon-<tenantname>.goskope.com_** — _Configuration download_ for the Netskope Client (Management Plane). It downloads the user's config files and also _auto-detects local explicit proxies_ to enable interoperate mode.
- **_gateway.gslb.goskope.com_** — _GSLB (Global Server Load Balancing) API endpoint_. The Netskope Client calls this to perform latency-based Data Plane selection — it returns the list of nearby data centers so the client connects to the optimal DP POP.
- **_achecker-<tenantname>.goskope.com_** — _Client enforcement_ (Management Plane). It checks for the presence of other Netskope steering methods (GRE/IPSec) and maintains an HTTP longpoll connection for _Client notifications/popups_. If the Client isn't installed, users hitting a steered app get redirected to an install page via this domain.
