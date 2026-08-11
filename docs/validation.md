# Network Validation

## Overview

This document validates the functionality of the Enterprise LAN network after completing VLAN segmentation, 802.1Q trunking, Router-on-a-Stick inter-VLAN routing, centralized DHCP, and end-to-end connectivity.

The validation process confirms that:

* VLANs are correctly segmented.
* Trunk links are operational.
* Router subinterfaces are functioning.
* DHCP is assigning addresses to clients.
* DHCP relay is forwarding client broadcasts to the centralized DHCP server.
* Hosts can communicate with their default gateways.
* Inter-VLAN routing is working.
* All VLANs can reach the centralized DHCP server network.
* End-to-end communication between different VLANs is successful.

---

## 1. Validation Environment

### VLANs

| VLAN           | Department  | Network            | Gateway         |
| -------------- | ----------- | ------------------ | --------------- |
| VLAN 10        | HR          | `192.168.10.0/24`  | `192.168.10.1`  |
| VLAN 20        | Sales       | `192.168.20.0/24`  | `192.168.20.1`  |
| VLAN 30        | IT          | `192.168.30.0/24`  | `192.168.30.1`  |
| Server Network | DHCP Server | `192.168.100.0/24` | `192.168.100.1` |

### DHCP Server

* IP Address: `192.168.100.10`
* Subnet Mask: `255.255.255.0`
* Default Gateway: `192.168.100.1`

---

# 2. Router Interface Validation

The router uses Router-on-a-Stick to provide Layer 3 gateways for the VLANs.

### Command

```cisco
show ip interface brief
```

### Expected Result

```text
GigabitEthernet0/0       unassigned      up    up
GigabitEthernet0/0.10    192.168.10.1    up    up
GigabitEthernet0/0.20    192.168.20.1    up    up
GigabitEthernet0/0.30    192.168.30.1    up    up
GigabitEthernet0/1       192.168.100.1   up    up
```

### Validation

All required router interfaces and subinterfaces are operational.

This confirms:

* VLAN 10 gateway is operational.
* VLAN 20 gateway is operational.
* VLAN 30 gateway is operational.
* Server network gateway is operational.
* Router-to-switch trunk interface is operational.

**Result: PASS**

---

# 3. VLAN Validation

The switch VLAN database should contain the required VLANs.

### Command

```cisco
show vlan brief
```

### Expected VLANs

```text
10    HR
20    SALES
30    IT
```

The corresponding access ports should be assigned to their correct VLANs.

### Validation

* HR PCs are connected to VLAN 10.
* Sales PCs are connected to VLAN 20.
* IT PCs are connected to VLAN 30.

**Result: PASS**

---

# 4. Trunk Validation

The links between the router and switches, and between the switches, use 802.1Q trunking.

### Command

```cisco
show interfaces gigabitEthernet0/1 switchport
```

### Expected Result

```text
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Trunking VLANs Enabled: 10,20,30
```

### Validation

The trunk is operational and allows:

* VLAN 10
* VLAN 20
* VLAN 30

This confirms that VLAN-tagged traffic can travel across the trunk links.

**Result: PASS**

---

# 5. DHCP Server Validation

The centralized DHCP server provides IP addresses for all three user VLANs.

### DHCP Pools

#### HR

```text
Pool Name: HR
Network: 192.168.10.0/24
Default Gateway: 192.168.10.1
Starting Address: 192.168.10.10
```

#### SALES

```text
Pool Name: SALES
Network: 192.168.20.0/24
Default Gateway: 192.168.20.1
Starting Address: 192.168.20.10
```

#### IT

```text
Pool Name: IT
Network: 192.168.30.0/24
Default Gateway: 192.168.30.1
Starting Address: 192.168.30.10
```

### Validation

The DHCP server successfully provides addresses to clients in all three VLANs.

Example assigned addresses:

```text
HR-PC1  → 192.168.10.10
SA-PC1  → 192.168.20.10
IT-PC1  → 192.168.30.10
```

**Result: PASS**

---

# 6. DHCP Relay Validation

Because the DHCP server is located in a different network, DHCP relay is configured on the router subinterfaces.

### Configuration

```cisco
interface GigabitEthernet0/0.10
 ip helper-address 192.168.100.10

interface GigabitEthernet0/0.20
 ip helper-address 192.168.100.10

interface GigabitEthernet0/0.30
 ip helper-address 192.168.100.10
```

### Validation

DHCP requests from:

* VLAN 10
* VLAN 20
* VLAN 30

are successfully forwarded to the centralized DHCP server at:

```text
192.168.100.10
```

**Result: PASS**

---

# 7. Default Gateway Connectivity

Each client must be able to communicate with its VLAN gateway.

### HR-PC1

```text
ping 192.168.10.1
```

Expected:

```text
Reply from 192.168.10.1
```

**Result: PASS**

### SA-PC1

```text
ping 192.168.20.1
```

Expected:

```text
Reply from 192.168.20.1
```

**Result: PASS**

### IT-PC1

```text
ping 192.168.30.1
```

Expected:

```text
Reply from 192.168.30.1
```

**Result: PASS**

---

# 8. Inter-VLAN Routing Validation

The router must route traffic between the three VLANs.

### Test 1 — HR to Sales

From HR-PC1:

```text
ping 192.168.20.10
```

Expected:

```text
Reply from 192.168.20.10
```

**Result: PASS**

### Test 2 — HR to IT

From HR-PC1:

```text
ping 192.168.30.10
```

Expected:

```text
Reply from 192.168.30.10
```

**Result: PASS**

### Test 3 — Sales to IT

From SA-PC1:

```text
ping 192.168.30.10
```

Expected:

```text
Reply from 192.168.30.10
```

**Result: PASS**

These tests confirm that Router-on-a-Stick is successfully providing inter-VLAN routing.

---

# 9. VLAN-to-DHCP Server Connectivity

All user VLANs must be able to communicate with the centralized DHCP server.

### HR → DHCP Server

From HR-PC1:

```text
ping 192.168.100.10
```

Result:

```text
3/4 or 4/4 replies
```

The first packet may occasionally time out because of initial ARP resolution.

**Result: PASS**

### Sales → DHCP Server

From SA-PC1:

```text
ping 192.168.100.10
```

Result:

```text
4/4 replies
```

**Result: PASS**

### IT → DHCP Server

From IT-PC1:

```text
ping 192.168.100.10
```

Result:

```text
4/4 replies
```

**Result: PASS**

---

# 10. End-to-End Connectivity

The final validation confirms communication across the complete network path:

```text
PC
 ↓
Access Switch
 ↓
Trunk
 ↓
Router
 ↓
Inter-VLAN Routing
 ↓
Destination VLAN / Server Network
 ↓
Destination Host
```

Successful tests include:

```text
HR-PC1 → SA-PC1
HR-PC1 → IT-PC1
SA-PC1 → IT-PC1
HR-PC1 → DHCP Server
SA-PC1 → DHCP Server
IT-PC1 → DHCP Server
```

### Validation Result

All tested networks successfully communicate through the router.

**Overall Result: PASS**

---

# 11. Validation Summary

| Test                | Expected Result               | Status |
| ------------------- | ----------------------------- | ------ |
| Router interfaces   | All required interfaces up/up | PASS   |
| VLAN 10             | Operational                   | PASS   |
| VLAN 20             | Operational                   | PASS   |
| VLAN 30             | Operational                   | PASS   |
| 802.1Q trunking     | Operational                   | PASS   |
| DHCP — HR           | Address assigned              | PASS   |
| DHCP — Sales        | Address assigned              | PASS   |
| DHCP — IT           | Address assigned              | PASS   |
| DHCP Relay          | Requests forwarded            | PASS   |
| HR → Gateway        | Reachable                     | PASS   |
| Sales → Gateway     | Reachable                     | PASS   |
| IT → Gateway        | Reachable                     | PASS   |
| HR → Sales          | Reachable                     | PASS   |
| HR → IT             | Reachable                     | PASS   |
| Sales → IT          | Reachable                     | PASS   |
| HR → DHCP Server    | Reachable                     | PASS   |
| Sales → DHCP Server | Reachable                     | PASS   |
| IT → DHCP Server    | Reachable                     | PASS   |

---

# 12. Final Validation Status

The Enterprise LAN implementation has successfully passed the required functional validation.

The network demonstrates:

* Proper VLAN segmentation
* 802.1Q trunking
* Router-on-a-Stick inter-VLAN routing
* Centralized DHCP
* DHCP relay
* Default gateway connectivity
* Inter-VLAN communication
* Server-network connectivity
* End-to-end network reachability

The implementation is therefore considered **functionally operational** in the Packet Tracer environment.
