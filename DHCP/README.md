# Packet Tracer: Centralized DHCP & Relay Configuration

## 1\. Project Overview

This simulation sets up a network where a single router (`R1`) manages IP address assignments for multiple networks.

  * **Site A (Left):** Connected to R1. Devices get IPs directly from R1.
  * **Site B (Right):** Connected to R2. Devices send broadcast requests, which R2 converts to unicast and forwards to R1.

## 2\. Network Topology

The network consists of two routers connected via a Serial WAN link using the `10.0.0.0/8` network.

![Network Topology Diagram](images/)

## 3\. Key Concepts

  * **DHCP Server (R1):** Holds the pools of IP addresses for both networks.
  * **DHCP Relay Agent (R2):** Uses the `ip helper-address` command. This intercepts UDP Broadcasts (like DHCP Discoveries) on the LAN interface and forwards them as Unicasts to the specific server IP.
  * **RIPv2 Routing:** Used to ensure R1 and R2 can route packets between the different subnets.

To visualize how R2 helps PC2 get an address from R1, see the process below:

## 4\. Configuration Details

### Router 1 (R1) - The DHCP Server

R1 is configured with **two** DHCP pools. One for its local LAN (`192.168.1.0`) and one for the remote LAN (`15.0.0.0`).

**Key Configuration Commands:**

```bash
! 1. Exclude IPs (Gateway and static devices)
ip dhcp excluded-address 192.168.1.1 192.168.1.9
ip dhcp excluded-address 15.0.0.1 15.0.0.9

! 2. Pool for Local LAN (Left Side)
ip dhcp pool ETHG0
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1

! 3. Pool for Remote LAN (Right Side)
ip dhcp pool SERS0
 network 15.0.0.0 255.0.0.0
 default-router 15.0.0.1
```

### Router 2 (R2) - The Relay Agent

R2 does not assign IPs itself (for the 15.0.0.0 network in this specific flow). Instead, it is configured to ask R1 for help using the `ip helper-address` command on the gateway interface.

**Key Configuration Commands:**

```bash
interface GigabitEthernet0/0/0
 ip address 15.0.0.1 255.0.0.0
 ! The command below forwards DHCP requests to R1
 ip helper-address 10.0.0.1
 duplex auto
 speed auto
```

## 5\. Addressing Table

| Device | Interface | IP Address | Subnet Mask | Role |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | Gig0/0/0 | 192.168.1.1 | 255.255.255.0 | Gateway for Site A |
| | Ser0/1/0 | 10.0.0.1 | 255.0.0.0 | WAN Connection |
| **R2** | Gig0/0/0 | 15.0.0.1 | 255.0.0.0 | Gateway for Site B |
| | Ser0/1/0 | 10.0.0.2 | 255.0.0.0 | WAN Connection |

## 6\. Verification

To verify the configuration works:

1.  **PC on Left (Site A):** Set to DHCP. It should receive an IP in the `192.168.1.x` range.
2.  **PC on Right (Site B):** Set to DHCP. It should receive an IP in the `15.x.x.x` range.
3.  **On Router 1:** Run `show ip dhcp binding` to see the leased addresses.

<!-- end list -->

```bash
R1# show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address      Client-ID/      Lease expiration        Type
                Hardware address/
                User name
192.168.1.10    0001.4321.ABCD  Mar 01 2002 01:30 PM    Automatic
15.0.0.10       0002.8765.DCBA  Mar 01 2002 01:35 PM    Automatic
```
