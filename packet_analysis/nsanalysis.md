---
layout: default
title: "Netskope Traffic Analysis"
permalink: /packet_analysis/nsanalysis/
---
# Mission

I will attempt to analyze my findings about all things Netskope, from a packet analysis perspective. These may consist in the analysis of traffic associated any Netskope product. The topics will be sorted from the most recent.

# Netskope GSLB Probing

## Motivation
When the Netskope agent is enabled, theory has it (and publicly available documentation as well) that the Netskope agent will start sending probes to the top-10 closest Netskope PoPs. How is this seen from a packet perspective?

The following small analysis was found in the same _Agent Enabled_ file.
## Findings

 1. A TLS connection to _gateway.gslb.goskope.com_, which is followed by the start of the TCP-probing the top-10 closest POPs. This means the client was requesting those top-10 POPs and subsequently probing them:

![[agent_enabled_images/tls2gslb_top10probe.png]]

2. After probing those POPs there is another TLS connection to _gateway.gslb.goskope.com_.

![[agent_enabled_images/tcp10probes_tls2gateway.png]]

3. Sometimes, I do not see anything else after that, other than this pattern repeating itself. However, there are times when this last TLS connection to _gateway.gslb.goskope.com_ appears  after the client probes those POPs:

![[agent_enabled_images/tlsPOP.png]]

4. Occasionally, I am seeing a TCP probe to _gateway.gslb.goskope.com_:

![[agent_enabled_images/tcpprobe2gslb.png]]

5. On another note, when analyzing their _iRTTs_, it is evident that the client starts with the closest PoP (as in, the PoP with the lowest probable round-trip time).

![[agent_enabled_images/top10pop_irtt.png]]


# Agent Enabled - Packet Flow

## Motivation
I had the curiosity of knowing how the Netskope agent works when enabled. There is a lot of publicly available documentation about the theory behind it, however, I wanted to find out some of that from a network-traffic perspective. 

A particular scenario I wanted to explore was about what traffic the Netskope agent was involved with when its tunnel was up. Most of all, when a user was not navigating to the Internet, as in, the Netskope agent is not tunneling any browser-based web traffic to the Netskope cloud. This means, what is the Netskope agent doing when the computer is running and the user has not interacted with the computer what-so-ever?

The following findings are centered around the Internet-based destinations the Netskope agent interacts with in this scenario.

## Requisites:
- The agent is already installed.
- The agent is user enabled.
- The user does not open any apps does not interact with anything in the computer.

## Netskope Findings:
These are the Netskope domains found in the Wireshark capture:

- As opposed to the **Agent Disabled** version of my analysis, more domains were seen.
- **How?**: `dns.qry.name matches "goskope" or tls matches "goskope"` also `dns.qry.name matches "infiot" or tls matches "infiot"`

| Domain                          | CNAME                             | IP                               | SNI                             |
| ------------------------------- | --------------------------------- | -------------------------------- | ------------------------------- |
| events.goskope.com              |                                   | 163.116.151.209, 163.116.128.104 | events.goskope.com              |
| addon-tenantname.goskope.com    | mgmt_lb.goskope.com               | 163.116.131.73                   | addon-tenantname.goskope.com    |
| gateway.gslb.goskope.com        |                                   | 163.116.128.79                   | gateway.gslb.goskope.com        |
| achecker-tenantname.goskope.com | achecker.mgmt_gateway.goskope.com | 163.116.131.156                  | achecker-tenantname.goskope.com |
| mgmt_gateway.epdlp.goskope.com  |                                   | 163.116.131.88                   | mgmt_gateway.epdlp.goskope.com  |
| tenantname.infiot.com           |                                   | 34.68.134.104                    | tenantname.infiot.net           |
|                                 |                                   | 163.116.234.87                   | mgmt_gateway.npa.goskope.com    |

When performing a _dig_ request on those domains, and subsequently on the _opskope_ and _infiot_ domains, the following NS servers are seen:

- **How?**: `dig @<recursive_server> <domain> +trace`

| goskope NS Servers      | opskope NS Server  | infiot NS Server              |
| ----------------------- | ------------------ | ----------------------------- |
| ns1.goskope.opskope.com | ns1glr.goskope.com | ns-cloud-e1.googledomains.com |
| ns2.goskope.opskope.com | ns2cvx.goskope.com | ns-cloud-e2.googledomains.com |
| ns3.goskope.com         | ns3ghw.goskope.com | ns-cloud-e3.googledomains.com |
| ns4.goskope.com         | ns4lny.goskope.com | ns-cloud-e4.googledomains.com |
| ns5.goskope.com         |                    |                               |
| ns6.goskope.com         |                    |                               |

For more context about those domains (please refer to https://docs.netskope.com):
- **_events.goskope.com_** — It collects telemetry/event data for monitoring user experience.
- **_addon-obritosalas.goskope.com_** — _Configuration download_ for the Netskope Client (Management Plane). It downloads the user's config files (`nsbranding.json`) and also _auto-detects local explicit proxies_ to enable interoperate mode.
- **_gateway.gslb.goskope.com_** — _GSLB (Global Server Load Balancing) API endpoint_. The Netskope Client calls this to perform latency-based Data Plane selection — it returns the list of nearby data centers so the client connects to the optimal DP POP.
- **_achecker-obritosalas.goskope.com_** — _Client enforcement_ (Management Plane). It checks for the presence of other Netskope steering methods (GRE/IPSec) and maintains an HTTP longpoll connection for _Client notifications/popups_. If the Client isn't installed, users hitting a steered app get redirected to an install page via this domain.
- **\*.epdlp.goskope.com**: The Endpoint DLP management plane gateway.
- **\*.infiot.com**: The BWAN gateway endpoint. The documentation is found in the official Netskope BWAN site.
- **\*.npa.goskope.com**: The Netskope Private Access enrollment endpoint for a particular tenant, homed at certain management plane locations. This endpoint is used by the Netskope client and Publisher for enrollment and Publisher registration, respectively. 


# Agent Disabled - Packet Flow
This is an attempt to analyze how the Netskope agent behaves when disabled, when the computer has no other applications opened. I ran a Wireshark capture and the following findings are what was seen in it.

## Motivation
A mystery in my head has been about what the Netskope agent does when disabled, meaning, the tunnel, that should steer traffic, is down. Theory has it that its processes are still running, which is true, which steers me to think that the agent is still contacting the Netskope cloud for something.

The following findings have to do with the destination the Netskope agent interacts with, at a basic level.

## Requisites
- The agent is already installed.
- The agent is user disabled.

## Netskope Findings
These are the Netskope domains found in the Wireshark capture:

- Display Filter `dns.qry.name matches "goskope"` 

| Domain                          | CNAME                                           | IP              | SNI                             |
| ------------------------------- | ----------------------------------------------- | --------------- | ------------------------------- |
| events.goskope.com              |                                                 | 163.116.131.96  | events.goskope.com              |
| addon-tenantname.goskope.com    | ingress-lbaas-addonman.mgmt_gateway.goskope.com | 163.116.131.73  | addon-tenantname.goskope.com    |
| gateway.gslb.goskope.com        |                                                 | 163.116.148.27  | gateway.gslb.goskope.com        |
| achecker-tenantname.goskope.com | achecker.mgmt_gateway.goskope.com               | 163.116.131.156 | achecker-tenantname.goskope.com |

When performing a _dig_ request on those domains, and subsequently on the _opskope_ domains, the following NS servers are seen:

- In a Linux CLI that aready has `dig`: `dig @<recursive_server> <domain> +trace`

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
- **_addon-tenantname.goskope.com_** — _Configuration download_ for the Netskope Client (Management Plane). It downloads the user's config files and also _auto-detects local explicit proxies_ to enable interoperate mode.
- **_gateway.gslb.goskope.com_** — _GSLB (Global Server Load Balancing) API endpoint_. The Netskope Client calls this to perform latency-based Data Plane selection — it returns the list of nearby data centers so the client connects to the optimal DP POP.
- **_achecker-tenantname.goskope.com_** — _Client enforcement_ (Management Plane). It checks for the presence of other Netskope steering methods (GRE/IPSec) and maintains an HTTP longpoll connection for _Client notifications/popups_. If the Client isn't installed, users hitting a steered app get redirected to an install page via this domain.
