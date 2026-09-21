
# CDP and LLDP Findings

I am currently re-studying the Cisco CCNA, and when learning about CDP and LLDP, a lightbulb lit up in my head... what would these two look in Wireshark? 

Since I had no way to simulate CDP and LLDP on the fly, I turned to [Johannes Weber's Ultimate PCAP](https://weberblog.net/the-ultimate-pcap/), and fortunately I found these traffic types, which I will analyze right now.

## Introduction

- _File_: The_Ultimate_PCAP.pcapng
- _Context_: Studying for the CCNA's CDP and LLDP.

## Cisco Discovery Protocol

- According to theory, CDP-talking devices advertise their information to other CDP-talking devices. This means CDP traffic provides information to the adjacent device.

- First thing I noticed, was the different stack in the packet:
	- **_Frame_**: The usual Frame information.
	- **_IEEE 802.3 Ethernet_**: Where the multicast destination MAC is found (_01:00:0c:cc:cc:cc_).
	- **_Logical-Link Control_**: Where, according to some investigation, is where Cisco managed to create its custom organization code (00:00:0c), and the CDP code (_0x2000_).
	- **_Cisco Discovery Protocol_**: Where the good stuff happens.

	![](ccna_images/cdp_packet_stack.png)

- In the CDP layer, I found several pieces of valuable information from the host advertising CDP. The following are things to note:
	- **Device ID_***: the device's name.
	- **_Software Version_**: the device's OS version.
	- **_Port ID_**: the device's interface, that sent the CDP. In this case, a FastEthernet0/1.
	- **_Platform_**: The device's product line (Cisco 3725)
	- **_Addresses_**: CDP also advertises IP addresses.
	- **_Capabilities_**: Can it be a router? A switch? A Host? Some of them? All of them?
	- **_Duplex_**: whether the device supports full / half duplex.

	![](ccna_images/cdp_advertised_info.png)

## Link-Layer Discovery Protocol

- The packet stack is different here, maybe because LLDP is a newer protocol than CDP:
	- **_Frame_**: The usual frame information.
	- **_Ethernet II_**: An Ethernet standard, a bit different from the IEEE 802.3. Here is where the multicast MAC address _01:80:c2:00:00:0e_ resides.
	- ***Link-Layer Discovery Protocol***: Where the good stuff happens.

- Some interesting I found in the LLDP-advertised information (the rest is similar from what was seen in the CDP use case):

	![](ccna_images/lldp_packet_stack.png)
	
	- **_Management Address_**: LLDP is able to send both the IPv4 and IPv6 addresses, if available.

		![](ccna_images/lldp_ipv6_mgmt.png)
		
	- **IEEE 802.3 - MAC/PHY Configuration/Status:** It has the different physical medium names with their supported duplex modes. If more than 1 is selected, it means the interface will be capable of supporting _auto-negotiation_.

		![](ccna_images/lldp_macphy_configs.png)

	- **_IEEE 802.3 - Power Via MDI:_** This presents the supported PoE parameters.

		![](ccna_images/lldp_poe_configs.png)

	- **_End of LLDPDU_**: The end of the LLDP packet.

## Conclusions

- Both CDP and LLDP advertise a devices information. The information might be different.

- CDP and LLDP, in general, use different lower-level stacks for the physical medium.

- These traffic types don't require an acknowledgement or response. They just advertise.