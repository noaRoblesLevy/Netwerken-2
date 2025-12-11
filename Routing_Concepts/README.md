# Packet Tracer Simulation: Router-on-a-Stick (Inter-VLAN Routing)

## 1\. Project Overview

This Cisco Packet Tracer simulation demonstrates a **Router-on-a-Stick** topology. This network design allows a single router interface to route traffic between multiple VLANs on a connected switch. This is achieved by creating virtual subinterfaces on the router and using 802.1Q encapsulation (Trunking).

## 2\. Topology Description

The network is divided into two distinct sites connected via a Serial WAN link:

  * **Left Site (Site A):**
      * **Router:** `Router0` (Gateway for local VLANs).
      * **Switch:** `Switch0` (Layer 2 connectivity).
      * **Devices:** `PC0` and `Laptop0`.
  * **Right Site (Site B):**
      * **Router:** `Router2` (Gateway for local VLANs).
      * **Switch:** `Switch1` (Layer 2 connectivity).
      * **Devices:** `PC1` and `PC2`.
  * **WAN Connection:**
      * A Serial connection links `Router0` and `Router2` to simulate a Wide Area Network.

## 3\. Key Concepts Covered

  * **VLANs (Virtual LANs):** Logical segmentation of a Layer 2 network (e.g., separating Sales from Engineering).
  * **802.1Q Trunking:** A protocol that tags frames with their VLAN ID, allowing multiple VLANs to traverse a single link between the Switch and the Router.
  * **Subinterfaces:** Virtual interfaces created on a physical router port (e.g., `g0/0.10`), each acting as a Default Gateway for a specific VLAN.
  * **Inter-VLAN Routing:** The process of forwarding traffic from one VLAN to another.

## 4\. Configuration Guide

### A. Switch Configuration

The switch ports connecting to end devices (PCs) must be in **Access Mode**, while the port connecting to the Router must be in **Trunk Mode**.

```bash
! Example Configuration for Switch0 / Switch1
enable
configure terminal

! Create VLANs
vlan 10
 name STAFF
vlan 20
 name GUEST

! Configure Access Ports (connecting to PCs)
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
!
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20

! Configure Trunk Port (connecting to Router)
interface GigabitEthernet0/1
 switchport mode trunk
! (Optional) Specify native vlan if necessary
! switchport trunk native vlan 99
```

### B. Router Configuration (Router-on-a-Stick)

The router physical interface remains with no IP address. IP addresses are assigned to subinterfaces corresponding to the VLANs.

```bash
! Example Configuration for Router0 / Router2
enable
configure terminal

! Enable the physical interface (Do not add IP here)
interface GigabitEthernet0/0/0
 no shutdown

! Configure Subinterface for VLAN 10
interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 description Gateway_for_VLAN10

! Configure Subinterface for VLAN 20
interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 description Gateway_for_VLAN20
```

### C. WAN Configuration (Serial Link)

Based on the screenshot CLI (`Router2`), the serial links utilize the `10.0.0.0` network.

```bash
! Router2 Configuration (From Screenshot)
interface Serial0/1/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
```

## 5\. Addressing Table (Template)

| Device | Interface | IP Address | Subnet Mask | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Router0** | Gig0/0/0.10 | 192.168.10.1 | 255.255.255.0 | VLAN 10 Gateway |
| | Gig0/0/0.20 | 192.168.20.1 | 255.255.255.0 | VLAN 20 Gateway |
| | Se0/1/0 | 10.0.0.1 | 255.255.255.252 | WAN Link (DCE) |
| **Router2** | Gig0/0/0.10 | 172.16.10.1 | 255.255.255.0 | VLAN 10 Gateway |
| | Gig0/0/0.20 | 172.16.20.1 | 255.255.255.0 | VLAN 20 Gateway |
| | Se0/1/0 | 10.0.0.2 | 255.255.255.252 | WAN Link (DTE) |
| **PC0** | NIC | 192.168.10.10 | 255.255.255.0 | VLAN 10 Client |

## 6\. Troubleshooting Steps

If pings are failing between VLANs (e.g., PC0 cannot ping Laptop0):

1.  **Check Gateway:** Ensure PCs have the correct Default Gateway configured (it should match the Router subinterface IP).
2.  **Check Trunking:** Ensure the link between the Switch and Router is active and set to `mode trunk`.
3.  **Check Encapsulation:** Verify the router subinterface has the correct VLAN ID command: `encapsulation dot1q <vlan-id>`.
4.  **Routing:** If trying to ping across the red WAN link, ensure a routing protocol (OSPF/EIGRP) or Static Routes are configured on both routers.
