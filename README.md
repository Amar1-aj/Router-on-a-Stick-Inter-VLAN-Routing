# Router-on-a-Stick Inter-VLAN Routing

## Project Overview
This Cisco Packet Tracer project demonstrates **Router-on-a-Stick (ROAS)** inter-VLAN routing. A single physical router interface uses IEEE 802.1Q subinterfaces to provide communication between devices in separate VLANs.

## Objectives
- Create and configure VLANs for IT, HR, and network management.
- Assign switch access ports to the correct VLANs.
- Configure an 802.1Q trunk between the switch and router.
- Configure router subinterfaces to route traffic between VLANs.
- Set up a native VLAN and isolate unused switch ports.
- Verify connectivity between VLANs.

## Network Design

| Device | Interface / Ports | Purpose |
|---|---|---|
| S1 | Fa0/1–Fa0/6 | IT access ports, VLAN 2 |
| S1 | Fa0/7–Fa0/12 | HR access ports, VLAN 3 |
| S1 | G0/1 | Trunk to router R1 |
| S1 | Fa0/13–Fa0/24, G0/2 | Unused ports, VLAN 21, shutdown |
| R1 | G0/0/1 | Physical trunk interface |
| R1 | G0/0/1.2, .3, .15, .20 | 802.1Q subinterfaces |

**Physical connection:** Connect S1 G0/1 to R1 G0/0/1 using an appropriate Ethernet cable.

## VLAN and IPv4 Addressing Plan

| VLAN | Name | Network | Default Gateway |
|---:|---|---|---|
| 2 | IT | 192.168.2.0/24 | 192.168.2.1 |
| 3 | HR | 192.168.3.0/24 | 192.168.3.1 |
| 15 | Management | 192.168.15.0/24 | 192.168.15.1 |
| 20 | Native | No user subnet configured | — |
| 21 | Unused | No user subnet configured | — |

### PC addressing

| PC | IPv4 address | Subnet mask | Default gateway |
|---|---|---|---|
| IT-PC1 | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| IT-PC2 | 192.168.2.11 | 255.255.255.0 | 192.168.2.1 |
| HR-PC1 | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 |
| HR-PC2 | 192.168.3.11 | 255.255.255.0 | 192.168.3.1 |

## Switch S1 Configuration

```cisco
enable
configure terminal
hostname S1

vlan 2
 name IT
vlan 3
 name HR
vlan 15
 name Management
vlan 20
 name Native
vlan 21
 name Unused

interface range fastethernet 0/1-6
 switchport mode access
 switchport access vlan 2

interface range fastethernet 0/7-12
 switchport mode access
 switchport access vlan 3

interface gigabitethernet 0/1
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 20
 switchport trunk allowed vlan 2,3,15,20

interface range fastethernet 0/13-24, gigabitethernet 0/2
 description Unused Ports
 switchport mode access
 switchport access vlan 21
 shutdown

interface vlan 15
 ip address 192.168.15.254 255.255.255.0
 no shutdown

ip default-gateway 192.168.15.1
end
copy running-config startup-config
```

## Router R1 Configuration

```cisco
enable
configure terminal
hostname R1

interface gigabitethernet 0/0/1
 no shutdown

interface gigabitethernet 0/0/1.2
 encapsulation dot1Q 2
 ip address 192.168.2.1 255.255.255.0

interface gigabitethernet 0/0/1.3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0

interface gigabitethernet 0/0/1.15
 encapsulation dot1Q 15
 ip address 192.168.15.1 255.255.255.0

interface gigabitethernet 0/0/1.20
 encapsulation dot1Q 20 native

end
copy running-config startup-config
```

## Verification

Run these commands on the switch:

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
show running-config
```

Run these commands on the router:

```cisco
show ip interface brief
show running-config
```

From each PC, test:
- Ping its own VLAN gateway.
- Ping a PC in the other VLAN (for example, IT-PC1 to HR-PC1).
- Ping the management SVI address `192.168.15.254` if management access is intended.

## Important Configuration Notes
1. The management SVI needs a subnet mask. This documentation uses `192.168.15.254 255.255.255.0`.
2. Configure the HR router subinterface with `192.168.3.1/24`; it was supplied separately in the original notes and is included in the complete configuration above.
3. Set the PC default gateways exactly as shown in the addressing table.
4. VLAN 20 is the native VLAN on both ends of the trunk. The router subinterface uses `encapsulation dot1Q 20 native`.
5. The switch trunk allows VLANs 2, 3, 15, and 20. VLAN 21 is intentionally not carried over the trunk because it is reserved for unused access ports.
6. Router-on-a-Stick requires the router physical interface to be enabled. The IP addresses belong on the subinterfaces, not the parent physical interface.

## Skills Demonstrated
- VLAN creation and access-port assignment
- IEEE 802.1Q trunking and native VLAN configuration
- Router-on-a-Stick inter-VLAN routing
- IPv4 subnetting and default gateway configuration
- Switch management SVI configuration
- Unused-port shutdown and basic switch hardening
- Network connectivity troubleshooting

## Project Outcome
The design enables IT and HR devices in separate VLANs to communicate through router R1, provided the cabling, VLAN assignments, IP addressing, and default gateways are configured correctly.
