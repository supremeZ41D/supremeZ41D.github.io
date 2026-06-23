---
layout: default
title: "Throughput Topics"
permalink: /packet_analysis/throughput/
---

## Congestion Window Sighting

- **Congestion Window** sighting in the Stevens graph on one of the 20.42.1.1 TCP streams, with both its slow-start.
- In particular for the slow-start, the Sequence Numbers increase in single bulks of data, which will change after the first sighting of a Selective ACK.

![[Pasted image 20260602121714.png]]
- Around the 2.25 second mark, as seen in the Stevens graph, some Selective Acks are seen. This can be verified when analyzing the TCPTrace graph for the same stream. After that, the Sequence Numbers increase in a two-step fashion.

![[Pasted image 20260619074722.png]]
- After the SACKs, the 20.42.1.1's Sequence Numbers do not increase exponentially, rather, 20.42.1.1 sends a steady stream of data, that when zoomed out, it looks like it has a linear behavior.

![[Pasted image 20260619083554.png]]
- Thereafter, the linear increase of the Sequence Numbers changes its behavior. Before the 5-second-mark SACK, the increments were a two-step process, however, after the SACK the increments became a 3-step-process.

![[Pasted image 20260619084142.png]]

- What is even more impressive is that later on, around the 9.5 second mark, another SACK is seen, that may have cause the Sequence Number increments to become a 4-step process. 
- It is worth noting that these steps have smaller Sequence Number increments, as opposed to when before the first SACK instance. Having more "steps" in the data transfer makes transfers a bit slower, as opposed to having a single or 2-step bursts.

![[Pasted image 20260619093200.png]]
- Going further in the TCPTrace and Stevens graphs, it seems the steps get reduced from 4, to 3, to 2. This means the sender is trying to speed-up the data transfer, hopefully trying to bring it down to just a single burst per RTT.

![[Pasted image 20260619100748.png]]

![[Pasted image 20260619100923.png]]

- **Conclusions**:
	- The initial exponential slow-start appears.
	- The sender tries sending data in single bursts, increasing those bursts every RTT.
	- After every SACK block, the sender segments the data transfer into smaller bursts of traffic. These segments start with 2, the 3 after another SACK block, then 4 segments after the final SACK block seen.
	- Given no other SACK block appears, the sender tries decreasing the segments to a single burst of data.
