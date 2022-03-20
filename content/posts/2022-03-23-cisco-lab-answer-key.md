---
categories:
- Network
date: "2022-03-23"
description: The solve of the Cisco course's final exam
slug: cisco-lab-answer-key
tags:
- Cisco
- configure
title: The solve of the Cisco course's final exam
---

![](/images/packet_tracer_homework.PNG)

1. Study the scheme. 
2. Split the network into subnets, addressing 5.87.0.0/24. Each PC is on a separate network. 
3. Perform basic configuration of all network devices. Raise only SSH protocol for management. 
4. Configure a separate VLAN for each PC on the switches and forward them to the router. Enable Port Fast BPDU Guard on internal interfaces. Enable Rapid PVST. 
5. Configure the OSPF protocol for routers, not including networks on internal interfaces. 
6. Raise a floating static route for OSPF protocol redundancy. 
7. Configure the GRE 1 protocol for the R1 - R2 - R4 route, GRE 2 for the R1 – R3 - R4 route, the routing method for connecting GRE and LAN is OSPF 
8. Configure the ACL to manage all devices only with MGT1 PC (SSH). 
9. Configure all devices time and data transfer to Syslog server (Server0).
10. Transfer the image and start-up configuration to TFTP Server0. Configure backup downloading from TFTP images on routers.
11. Check the network operability in case of failure of the OSPF protocol.

### Answer Key

- [Cisco Packet Tracer project](/images/cisco_config/answer_key.pkt)
- [R1 startup configure file](/images/cisco_config/R1_startup-config.txt)
- [R2 startup configure file](/images/cisco_config/R2_startup-config.txt)
- [R3 startup configure file](/images/cisco_config/R3_startup-config.txt)
- [R4 startup configure file](/images/cisco_config/R4_startup-config.txt)
- [S1 startup configure file](/images/cisco_config/S1_startup-config.txt)
- [S1 startup configure file](/images/cisco_config/S1_startup-config.txt)
