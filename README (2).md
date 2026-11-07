# Inter-VLAN Routing Using Router-on-a-Stick

## 1. Project Overview

This Cisco Packet Tracer project demonstrates **Inter-VLAN Routing using the Router-on-a-Stick (ROAS) method**. The topology contains one router, two Layer 2 switches, and end devices in two VLANs: VLAN 10 (IT) and VLAN 20 (FINANCE).

The two switches are connected by a trunk link. The second switch is also connected to the router through a trunk link. Both VLANs are available across both switches.

## 2. What Is Router-on-a-Stick?

**Router-on-a-Stick is an Inter-VLAN Routing design in which one physical router interface is divided into multiple logical subinterfaces. Each subinterface is assigned to a VLAN using IEEE 802.1Q VLAN tagging and is given an IP address that acts as the default gateway for that VLAN.**

The router uses a single physical Ethernet connection to the switch, while the switch-to-router link operates as an 802.1Q trunk carrying traffic for multiple VLANs. The router receives tagged frames, identifies their VLAN using the 802.1Q tag, routes packets between the IP networks, and sends the frames back through the appropriate subinterface.

In this lab:
- `Fa0/0.10` handles VLAN 10.
- `Fa0/0.20` handles VLAN 20.
- `encapsulation dot1Q 10` associates subinterface `.10` with VLAN 10.
- `encapsulation dot1Q 20` associates subinterface `.20` with VLAN 20.

The `.10` and `.20` subinterface numbers are labels chosen by the administrator; the VLAN association is established by the `encapsulation dot1Q` command.

## 3. Why Is Inter-VLAN Routing Required?

A Layer 2 switch forwards frames within VLANs. VLANs are separate Layer 2 broadcast domains, and devices in different IP subnets need a Layer 3 device to route traffic between them.

Without Inter-VLAN Routing, a PC in VLAN 10 cannot communicate with a PC in VLAN 20 through normal IP routing. The router provides the Layer 3 forwarding between these networks.

## 4. Topology and Connections

| Device / Port | Connected To | Purpose |
|---|---|---|
| SW1 Fa0/5 | SW2 Fa0/5 | Trunk between switches |
| SW2 Fa0/6 | R1 Fa0/0 | Trunk between switch and router |
| SW1 Fa0/1-2 | PCs | VLAN 20 access ports |
| SW1 Fa0/3-4 | PCs | VLAN 10 access ports |
| SW2 Fa0/1-2 | PCs | VLAN 20 access ports |
| SW2 Fa0/3-4 | PCs | VLAN 10 access ports |

Both trunk links carry VLANs 10 and 20.

## 5. VLAN and IP Addressing Plan

| VLAN | Name | Network | Default Gateway |
|---|---|---|---|
| 10 | IT | `192.168.1.0/24` | `192.168.1.10` |
| 20 | FINANCE | `192.168.2.0/24` | `192.168.2.10` |

Example host addressing (each PC must have a unique IP):

| VLAN | Example PC IPs | Subnet Mask | Default Gateway |
|---|---|---|---|
| 10 | `192.168.1.2` to `192.168.1.5` | `255.255.255.0` | `192.168.1.10` |
| 20 | `192.168.2.2` to `192.168.2.5` | `255.255.255.0` | `192.168.2.10` |

## 6. Configuration

The complete device-by-device CLI commands are provided in [`commands.txt`](commands.txt).

Configuration stages:
1. Create VLAN 10 and VLAN 20 on both switches.
2. Assign PC-facing switch ports as access ports in the correct VLAN.
3. Configure the SW1–SW2 link as a trunk allowing VLANs 10 and 20.
4. Configure the SW2–router link as a trunk allowing VLANs 10 and 20.
5. Enable router interface `Fa0/0`.
6. Create router subinterfaces and associate each with its VLAN using 802.1Q.
7. Assign the matching IP address to each router subinterface.
8. Configure each PC's IP address, subnet mask, and default gateway.

## 7. How Traffic Is Routed Between VLANs

Example: a PC in VLAN 10 sends traffic to a PC in VLAN 20.

1. The source PC recognizes that the destination is in a different IP subnet.
2. It sends the packet to its default gateway, `192.168.1.10`.
3. The frame travels across the switch trunks with VLAN 10 identification.
4. Router subinterface `Fa0/0.10` receives the VLAN 10 traffic.
5. The router examines the destination IP address and routes the packet toward the VLAN 20 network.
6. The router sends the packet through subinterface `Fa0/0.20`, using VLAN 20 tagging on the trunk.
7. SW2 forwards the VLAN 20 frame toward the destination PC (across the inter-switch trunk if the destination is connected to SW1).

The return traffic follows the reverse Layer 3 path.

## 8. Verification Commands

### Switches

```cisco
show vlan brief
show interfaces trunk
show running-config
```

### Router

```cisco
show ip interface brief
show ip route
show running-config
```

Expected router interfaces should show:
- `Fa0/0` — up/up
- `Fa0/0.10` — `192.168.1.10`, up/up
- `Fa0/0.20` — `192.168.2.10`, up/up

The routing table should show both VLAN networks as directly connected:
- `192.168.1.0/24` via `Fa0/0.10`
- `192.168.2.0/24` via `Fa0/0.20`

## 9. Connectivity Tests

1. Ping the VLAN 10 gateway (`192.168.1.10`) from a VLAN 10 PC.
2. Ping the VLAN 20 gateway (`192.168.2.10`) from a VLAN 20 PC.
3. Ping a VLAN 20 PC from a VLAN 10 PC.
4. Ping a VLAN 10 PC from a VLAN 20 PC.
5. Test same-VLAN communication between PCs connected to different switches.

Successful cross-VLAN pings confirm that the router is routing between the two VLAN subnets.

## 10. Troubleshooting Checklist

- Confirm both VLANs exist on both switches.
- Confirm access ports are assigned to the intended VLAN.
- Confirm both ends of each trunk link are configured as trunks.
- Confirm VLANs 10 and 20 are allowed on both trunk links.
- Confirm router `Fa0/0` is enabled and operational.
- Confirm `encapsulation dot1Q` uses the correct VLAN IDs.
- Confirm router subinterface IP addresses match the PC default gateways.
- Confirm each PC has an IP address and subnet mask from its own VLAN subnet.
- Use `show interfaces trunk`, `show ip interface brief`, and `show ip route` to isolate the fault.

## 11. Key Learning Outcomes

- Creating and assigning VLANs.
- Configuring access and trunk ports.
- Understanding IEEE 802.1Q VLAN tagging.
- Understanding router physical interfaces and logical subinterfaces.
- Understanding default gateways and directly connected routes.
- Enabling communication between separate VLANs using Layer 3 routing.

## 12. Important Technical Notes

- Router-on-a-Stick uses one physical router interface for multiple VLANs by creating logical subinterfaces.
- The switch-to-router connection must be a trunk so it can carry traffic for multiple VLANs.
- The inter-switch link must also carry the VLANs that need to extend between switches.
- The router's physical interface normally has no IP address in this design; the gateway IP addresses are configured on its subinterfaces.
- The subinterface number does not itself create or identify the VLAN. The `encapsulation dot1Q <vlan-id>` command specifies the VLAN tag.
- On some Cisco switch models, trunk encapsulation is fixed to 802.1Q, so the `switchport trunk encapsulation dot1q` command is not required or accepted.

## Result

The lab demonstrates how two VLANs distributed across two Layer 2 switches can communicate through a single router physical interface using Router-on-a-Stick Inter-VLAN Routing.
