# Secure Office Network Using VLANs and Inter-VLAN Routing

**Name:** Tanmay Sanghavi
**Subject:** Computer Networking
**Software:** Cisco Packet Tracer
**Project file:** `TanmaySanghavi_VLAN_Routing_Project.pkt`

---

## 1. About the Project

In this project I worked as a junior network administrator for a small company with three departments: HR, IT and Finance. All of them were on one flat network, so every broadcast reached every PC and nothing separated one department from another.

My job was to split the network into three VLANs spread across two switches, and use a single router (Router-on-a-Stick) so the departments can still reach each other when they need to.

---

## 2. Network Topology

![Network Topology](Screen_Shots/TestD_ARP_request.png)

### Devices used

| Device | Model | Quantity |
|---|---|---|
| Router | Cisco 2911 (R1) | 1 |
| Switches | Cisco 2960 (S1, S2) | 2 |
| PCs | HR-PC1, HR-PC2, IT-PC1, IT-PC2, FIN-PC1, FIN-PC2 | 6 |

### Connections

| From | Port | To | Port | Cable |
|---|---|---|---|---|
| R1 | G0/0 | S1 | G0/1 | Copper straight-through |
| S1 | G0/2 | S2 | G0/1 | Copper cross-over |
| HR-PC1 | Fa0 | S1 | Fa0/1 | Copper straight-through |
| IT-PC1 | Fa0 | S1 | Fa0/2 | Copper straight-through |
| FIN-PC1 | Fa0 | S1 | Fa0/3 | Copper straight-through |
| HR-PC2 | Fa0 | S2 | Fa0/1 | Copper straight-through |
| IT-PC2 | Fa0 | S2 | Fa0/2 | Copper straight-through |
| FIN-PC2 | Fa0 | S2 | Fa0/3 | Copper straight-through |

I put one PC from each department on each switch, so every department is spread across both switches. This shows that one VLAN can stretch across more than one switch.

---

## 3. VLAN Plan

| VLAN ID | Name | Department | Network |
|---|---|---|---|
| 10 | HR | Human Resources | 192.168.10.0/24 |
| 20 | IT | Information Technology | 192.168.20.0/24 |
| 30 | FINANCE | Finance | 192.168.30.0/24 |

On both switches, Fa0/1 is HR, Fa0/2 is IT and Fa0/3 is Finance.

---

## 4. IP Addressing

| Device | VLAN | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| HR-PC1 | 10 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| HR-PC2 | 10 | 192.168.10.12 | 255.255.255.0 | 192.168.10.1 |
| IT-PC1 | 20 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |
| IT-PC2 | 20 | 192.168.20.12 | 255.255.255.0 | 192.168.20.1 |
| FIN-PC1 | 30 | 192.168.30.11 | 255.255.255.0 | 192.168.30.1 |
| FIN-PC2 | 30 | 192.168.30.12 | 255.255.255.0 | 192.168.30.1 |

| Router Subinterface | VLAN | IP Address |
|---|---|---|
| G0/0.10 | 10 | 192.168.10.1/24 |
| G0/0.20 | 20 | 192.168.20.1/24 |
| G0/0.30 | 30 | 192.168.30.1/24 |

---

## 5. Configuration

### Switch S1

```
enable
configure terminal
hostname S1

vlan 10
 name HR
vlan 20
 name IT
vlan 30
 name FINANCE
exit

interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface fa0/3
 switchport mode access
 switchport access vlan 30

interface g0/1
 switchport mode trunk
interface g0/2
 switchport mode trunk
end
copy running-config startup-config
```

### Switch S2

```
enable
configure terminal
hostname S2

vlan 10
 name HR
vlan 20
 name IT
vlan 30
 name FINANCE
exit

interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface fa0/3
 switchport mode access
 switchport access vlan 30

interface g0/1
 switchport mode trunk
end
copy running-config startup-config
```

### Router R1 (Router-on-a-Stick)

```
enable
configure terminal
hostname R1

interface g0/0
 no shutdown
exit

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit
interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
interface g0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
end
copy running-config startup-config
```

**What I did and why:**

- **Created the VLANs on both switches.** A switch has to know about a VLAN before it can carry it. HR-PC2 sits on S2, so without VLAN 10 on S2 it couldn't be part of HR.
- **Set the PC ports as access ports.** An access port belongs to just one VLAN, which is how each PC ends up in its department.
- **Set the switch-to-switch and switch-to-router links as trunks.** A trunk carries every VLAN over one cable. Each frame gets an 802.1Q tag so the device on the other end knows which VLAN it belongs to.
- **Turned on the router port with `no shutdown`.** Router ports are off by default.
- **Created one subinterface per VLAN.** The router has only one cable to S1, so I split G0/0 into three logical subinterfaces. `encapsulation dot1Q` tells each one which VLAN tag it handles, and its IP address is the default gateway for that department.

---

## 6. Verification

### Switch commands

| Command | What it shows |
|---|---|
| `show vlan brief` | All VLANs and which access ports are in each one |
| `show interfaces trunk` | Which ports are trunks, their encapsulation (802.1Q) and which VLANs they carry |
| `show interfaces status` | Each port's status (connected or not), VLAN, speed and duplex |
| `show mac address-table` | Which MAC addresses the switch has learned, and on which port and VLAN |
| `show running-config` | The full current configuration of the switch |

**S1 – show vlan brief**

Fa0/1 is in HR, Fa0/2 in IT and Fa0/3 in FINANCE.

![S1 VLAN brief](Screen_Shots/S1_vlan_brief.png)

**S2 – show vlan brief**

Same port layout as S1.

![S2 VLAN brief](Screen_Shots/S2_vlan_brief.png)

**S1 – show interfaces trunk (before the router was turned on)**

Only Gig0/2 (the link to S2) shows up here. This command only lists trunks that are up, and the router port was still shut down at this point.

![S1 trunk before router](Screen_Shots/S1_show_interfaces_trunk.png)

**S1 – show interfaces trunk (after the router was turned on)**

Both Gig0/1 (to R1) and Gig0/2 (to S2) are now trunking with 802.1Q and carrying VLANs 1, 10, 20 and 30.

![S1 trunk after router](Screen_Shots/v2_S1_show_interfaces_trunk.png)

**S2 – show interfaces trunk**

Gig0/1 (to S1) is trunking and carries VLANs 1, 10, 20 and 30.

![S2 trunk](Screen_Shots/S2_show_interfaces_trunk.png.png)

### Router commands

| Command | What it shows |
|---|---|
| `show ip interface brief` | All interfaces with their IP addresses and up/down status |
| `show ip route` | The routing table. "C" is a directly connected network and "L" is the router's own address on it |
| `show running-config` | The full router configuration, including the subinterfaces |
| `show interfaces` | Detailed info per interface, like MAC address, encapsulation, packet counts and errors |

**R1 – show ip interface brief**

G0/0 and all three subinterfaces are up/up.

![R1 ip interface brief](Screen_Shots/R1_show_ip_interface_brief.png)

**R1 – show ip route**

The router has a connected route for each VLAN network: 192.168.10.0, 192.168.20.0 and 192.168.30.0.

![R1 ip route](Screen_Shots/R1_show_ip_route.png)

**R1 – show running-config (subinterfaces)**

![R1 running config](Screen_Shots/R1_running_config.png)

---

## 7. Testing

### Test A: Same-VLAN Communication

| From | To | Result |
|---|---|---|
| HR-PC1 | HR-PC2 (192.168.10.12) | Success |
| IT-PC1 | IT-PC2 (192.168.20.12) | Success |
| FIN-PC1 | FIN-PC2 (192.168.30.12) | Success |

These PCs are in the same VLAN but on different switches, so the traffic went from S1 to S2 over the trunk. The router wasn't involved.

### Test B: Inter-VLAN Communication

| From | To | Result |
|---|---|---|
| HR-PC1 | IT-PC1 (192.168.20.11) | Success, 4/4 replies |
| HR-PC1 | FIN-PC1 (192.168.30.11) | Success, 4/4 replies |
| FIN-PC1 | HR-PC1 (192.168.10.11) | Success, 4/4 replies |

These PCs are in different VLANs, so the traffic had to go through the router. Every reply shows **TTL=127**. A Windows PC starts at 128 and the TTL drops by one at each router, so 127 confirms the packets passed through R1.

**HR-PC1 → IT-PC1**
![HR-PC1 to IT-PC1](Screen_Shots/HR-PC1_to_IT-PCI.png)

**HR-PC1 → FIN-PC1**
![HR-PC1 to FIN-PC1](Screen_Shots/HR-PC1_to_FIN-PC1.png)

**FIN-PC1 → HR-PC1**
![FIN-PC1 to HR-PC1](Screen_Shots/FIN-PC1_to_HR-PC1.png)

### Test C: VLAN Isolation

I shut down the router's G0/0 interface. As the output shows, G0/0 and all three subinterfaces went **administratively down**.

![Router shut down](Screen_Shots/TestC_R1_shutdown.png)

**1. Can two PCs in the same VLAN still communicate?**
Yes. HR-PC1 could still ping HR-PC2. Traffic inside one VLAN is handled by the switches alone and travels between S1 and S2 over the trunk.

**2. Can a PC in VLAN 10 communicate with a PC in VLAN 20?**
No. The ping from HR-PC1 to IT-PC1 timed out.

**3. Why does the router affect communication between VLANs but not within the same VLAN?**
Each VLAN is its own network. When a PC sends data to another PC on the same network, the switches deliver it directly using MAC addresses. When the destination is on a different network, the PC has to send the data to its default gateway, and that gateway is a subinterface on the router. With the router link down, the gateway can't be reached, so traffic between VLANs stops. Traffic within a VLAN never needs the gateway, so it keeps working.

After the test I brought G0/0 back up with `no shutdown`. The interface and subinterfaces came back up and inter-VLAN pings worked again.

![Router restored](Screen_Shots/TestC_router_restored.png)

### Test D: Simulation Mode

I switched to Simulation Mode with only the ARP and ICMP filters on, and sent a ping from HR-PC1.

**ARP request leaving HR-PC1**
![ARP request](Screen_Shots/TestD_ARP_request.png)

**Frame at R1 with the 802.1Q (Dot1q) header**

The PDU details at R1 show the frame arriving on GigabitEthernet0/0 with a Dot1q header. On the way in, the source MAC is HR-PC1 (0060.3E8C.4323) and the destination MAC is R1 (00E0.F972.3401). On the way out, they're swapped. The IP addresses stay the same throughout.

![Dot1q tag at R1](Screen_Shots/TestD_dot1q_tag.png)

**1. How does the source PC find its default gateway's MAC address?**
HR-PC1 compares the destination IP with its own network and sees that it's on a different network, so the packet has to go to the default gateway, 192.168.10.1. Its ARP table doesn't have the gateway's MAC yet, so it sends an ARP request as a broadcast inside VLAN 10, asking who has 192.168.10.1. The router's G0/0.10 subinterface replies with its MAC address.

**2. How does the Ethernet frame travel from the access switch to the router?**
The frame leaves HR-PC1 without a VLAN tag. S1 receives it on Fa0/1, an access port in VLAN 10. S1 adds an 802.1Q tag for VLAN 10 and sends it out of trunk port G0/1 to the router.

**3. How does the router identify the correct outgoing interface?**
The frame arrives tagged VLAN 10, so subinterface G0/0.10 accepts it. The router removes the tag, reads the destination IP address and checks its routing table. For a packet going to FIN-PC1 (192.168.30.11), the table shows 192.168.30.0/24 directly connected on G0/0.30, so the router sends it out of that subinterface with a VLAN 30 tag.

**4. What happens to the source and destination MAC addresses?**
They change at the router. Before the router, the source MAC is HR-PC1 and the destination MAC is the router. After the router, the source MAC is the router and the destination MAC is the destination PC.

**5. Do the source and destination IP addresses change?**
No. They stay the same for the whole trip because there's no NAT in this network. Only the TTL goes down by one at the router.

---

## 8. Troubleshooting Report

### Issues I ran into while building the network

| Issue | What I saw | Cause | Fix |
|---|---|---|---|
| Router link stayed red | The R1–S1 cable had red lights right after cabling | Router interfaces are shut down by default | Ran `no shutdown` on G0/0 |
| S1's trunk to the router wasn't listed | `show interfaces trunk` on S1 only showed Gig0/2 | The command only lists trunks that are up, and the router port was still off | After turning on G0/0, Gig0/1 showed as trunking (see the second S1 trunk screenshot) |
| First ping sometimes timed out | The first reply said "Request timed out" | The PC and router were still learning MAC addresses through ARP | Ran the ping again and got all replies |

### Other common faults in this kind of setup

| Fault | Symptom | How to find it | Fix |
|---|---|---|---|
| Switch-to-switch link not a trunk | PCs in the same VLAN on different switches can't ping each other | `show interfaces trunk` doesn't list the port | `switchport mode trunk` on both ends |
| PC port in the wrong VLAN | A PC can't reach its own department | `show vlan brief` shows the port in the wrong VLAN | `switchport access vlan <correct VLAN>` |
| VLAN missing on one switch | PCs of that VLAN on that switch are cut off | `show vlan brief` doesn't list the VLAN | Create the VLAN on that switch |
| Wrong VLAN number in encapsulation | One department can't reach the others | `show running-config` on R1 | Correct the `encapsulation dot1Q` value |
| `ip address` entered before `encapsulation` | The router gives an error on the subinterface | Error message in the CLI | Enter `encapsulation dot1Q` first, then the IP |
| Wrong default gateway on a PC | Same-VLAN pings work but inter-VLAN pings fail | `ipconfig` on the PC | Set the gateway to the .1 address of its VLAN |

---

## 9. Conclusion

This project showed me how VLANs make a network easier to manage and more secure. Splitting the office into HR, IT and Finance VLANs gave each department its own broadcast domain, so broadcasts stay inside one department and the departments are kept apart even though they share the same switches. Trunk links let one cable carry all three VLANs using 802.1Q tags, and Router-on-a-Stick let the departments talk to each other through a single router interface when needed. The tests confirmed that traffic within a VLAN only needs the switches, while traffic between VLANs depends on the router.

---

## 10. Files in this Repository

| File | Description |
|---|---|
| `TanmaySanghavi_VLAN_Routing_Project.pkt` | Cisco Packet Tracer project file |
| `README.md` | This report |
| `Screen_Shots/` | All screenshots used in this report |
