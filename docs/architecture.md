# Network Architecture

## 1. Overview

This project implements a small enterprise campus network using Cisco routing and switching technologies.

The primary objective is to provide:

* Department-based VLAN segmentation
* Layer 2 switching across multiple switches
* 802.1Q trunking
* Inter-VLAN routing using Router-on-a-Stick
* Centralized DHCP services
* DHCP relay using `ip helper-address`
* Dedicated server network
* End-to-end network connectivity and validation

The network separates users into three departmental VLANs:

* VLAN 10 — HR
* VLAN 20 — SALES
* VLAN 30 — IT

A dedicated server network is used for infrastructure services such as DHCP.

---

## 2. High-Level Topology

```text
                         ┌──────────────────────┐
                         │     DHCP Server      │
                         │    192.168.100.10    │
                         │   Server Network     │
                         └──────────┬───────────┘
                                    │
                              192.168.100.0/24
                                    │
                             ┌──────┴──────┐
                             │     R1      │
                             │ Cisco 2911  │
                             │             │
                             │ Router-on-  │
                             │   a-Stick   │
                             └──────┬──────┘
                                    │
                              802.1Q Trunk
                                    │
                         ┌──────────┴──────────┐
                         │         SW1         │
                         │ Cisco Catalyst      │
                         │     3560-24PS       │
                         └──────────┬──────────┘
                                    │
                              802.1Q Trunk
                                    │
                         ┌──────────┴──────────┐
                         │         SW2         │
                         │ Cisco Catalyst      │
                         │        2960        │
                         └──────┬────┬────┬───┘
                                │    │    │
                              VLAN10 VLAN20 VLAN30
                                │    │    │
                               HR  SALES  IT
```

---

## 3. Network Devices

| Device      | Model                    | Role                                             |
| ----------- | ------------------------ | ------------------------------------------------ |
| R1          | Cisco 2911               | Inter-VLAN routing, default gateways, DHCP relay |
| SW1         | Cisco Catalyst 3560-24PS | Main Layer 2 switching / distribution switch     |
| SW2         | Cisco Catalyst 2960      | Access switch                                    |
| DHCP Server | Generic Server           | Centralized DHCP service                         |
| HR PCs      | Generic PC               | VLAN 10 clients                                  |
| SALES PCs   | Generic PC               | VLAN 20 clients                                  |
| IT PCs      | Generic PC               | VLAN 30 clients                                  |

### Switch Roles

**SW1 — Cisco Catalyst 3560-24PS**

SW1 acts as the primary switch in the topology and provides Layer 2 connectivity between R1 and SW2.

Although the Catalyst 3560 supports Layer 3 switching capabilities, this project intentionally uses R1 for Inter-VLAN Routing through Router-on-a-Stick. This keeps the lab focused on 802.1Q trunking, router subinterfaces, and centralized routing.

**SW2 — Cisco Catalyst 2960**

SW2 operates as the access-layer switch and connects the departmental end devices to the VLAN infrastructure.

---

## 4. VLAN Architecture

The network uses VLANs to logically separate users according to department.

### VLAN 10 — HR

```text
VLAN ID:       10
Name:          HR
Network:       192.168.10.0/24
Gateway:       192.168.10.1
```

HR users are isolated at Layer 2 from the SALES and IT broadcast domains.

---

### VLAN 20 — SALES

```text
VLAN ID:       20
Name:          SALES
Network:       192.168.20.0/24
Gateway:       192.168.20.1
```

SALES users operate within their own Layer 2 broadcast domain.

---

### VLAN 30 — IT

```text
VLAN ID:       30
Name:          IT
Network:       192.168.30.0/24
Gateway:       192.168.30.1
```

IT users operate within a separate broadcast domain.

---

### Server Network

Infrastructure services are placed in a separate network:

```text
Network:       192.168.100.0/24
Gateway:       192.168.100.1
DHCP Server:   192.168.100.10
```

This separates infrastructure servers from departmental client VLANs.

---

## 5. Router-on-a-Stick Architecture

R1 uses a single physical interface, `GigabitEthernet0/0`, to route traffic for multiple VLANs.

The physical interface itself does not have an IP address.

Instead, logical subinterfaces are created:

```text
GigabitEthernet0/0.10
        │
        └── VLAN 10
            192.168.10.1

GigabitEthernet0/0.20
        │
        └── VLAN 20
            192.168.20.1

GigabitEthernet0/0.30
        │
        └── VLAN 30
            192.168.30.1
```

Each subinterface uses IEEE 802.1Q encapsulation to identify the VLAN associated with incoming and outgoing frames.

This allows a single physical router interface to provide Layer 3 gateways for multiple VLANs.

---

## 6. Trunk Architecture

The links between the router and switching infrastructure carry multiple VLANs simultaneously.

The network uses IEEE 802.1Q trunking.

### R1 ↔ SW1

The trunk between R1 and SW1 carries:

```text
VLAN 10
VLAN 20
VLAN 30
```

This allows R1 to receive and route traffic from all departmental VLANs through its `GigabitEthernet0/0` subinterfaces.

### SW1 ↔ SW2

The trunk between SW1 and SW2 also carries:

```text
VLAN 10
VLAN 20
VLAN 30
```

Therefore, VLAN traffic can traverse the switching infrastructure while maintaining its VLAN identity.

---

## 7. DHCP Architecture

DHCP is centralized on the server at:

```text
192.168.100.10
```

The DHCP server is not directly connected to each client VLAN.

Instead, R1 acts as a DHCP relay.

Each client VLAN has an `ip helper-address` pointing to the DHCP server:

```text
VLAN 10
192.168.10.1
      │
      └── DHCP Relay
          ↓
      192.168.100.10


VLAN 20
192.168.20.1
      │
      └── DHCP Relay
          ↓
      192.168.100.10


VLAN 30
192.168.30.1
      │
      └── DHCP Relay
          ↓
      192.168.100.10
```

This allows clients from all three VLANs to obtain their IP configuration from the centralized DHCP server.

---

## 8. Traffic Flow

### 8.1 Client DHCP Request

When a client starts without an IP address:

```text
Client
  │
  │ DHCP Broadcast
  ▼
Access Switch
  │
  │ VLAN-specific traffic
  ▼
R1 VLAN Subinterface
  │
  │ DHCP Relay
  ▼
DHCP Server
192.168.100.10
```

R1 forwards the DHCP request toward the centralized DHCP server.

The server then provides an address appropriate for the client's VLAN.

---

### 8.2 Inter-VLAN Communication

When an HR client communicates with a SALES client:

```text
HR PC
192.168.10.x
     │
     ▼
VLAN 10 Gateway
192.168.10.1
     │
     ▼
R1
     │
     ▼
VLAN 20 Gateway
192.168.20.1
     │
     ▼
SALES PC
192.168.20.x
```

The traffic must pass through R1 because VLAN 10 and VLAN 20 are separate Layer 2 broadcast domains.

The same principle applies to communication between VLAN 10 and VLAN 30 or VLAN 20 and VLAN 30.

---

## 9. Layered Architecture

The implementation can be understood using the following layers.

### Layer 2 — Switching

Responsible for:

* VLAN creation
* Access ports
* VLAN membership
* 802.1Q trunking
* MAC-based switching
* Broadcast-domain separation

SW1 and SW2 provide the Layer 2 switching infrastructure.

### Layer 3 — Routing

Responsible for:

* Default gateways
* Inter-VLAN routing
* Routing between departmental networks
* Connectivity to the server network

R1 provides these Layer 3 functions.

### Infrastructure Services

Responsible for:

* DHCP
* Centralized IP address allocation
* DHCP relay

The dedicated server network provides centralized infrastructure services.

---

## 10. Design Rationale

### Why use VLANs?

Without VLANs, all departmental devices would share the same Layer 2 broadcast domain.

VLAN segmentation provides:

* Smaller broadcast domains
* Logical department separation
* Better traffic organization
* A foundation for security policies
* Easier network management

---

### Why use two different switch platforms?

The lab intentionally uses two different Cisco switch models:

```text
SW1 → Cisco Catalyst 3560-24PS
SW2 → Cisco Catalyst 2960
```

This demonstrates that VLANs and 802.1Q trunking can operate across different Cisco switch platforms when compatible Layer 2 configurations are used.

It also makes the lab more representative of a mixed-device enterprise environment.

---

### Why use Router-on-a-Stick?

For this lab, Router-on-a-Stick provides a practical way to implement Inter-VLAN Routing using a single router interface.

It reduces hardware requirements while demonstrating:

* 802.1Q
* Router subinterfaces
* Inter-VLAN routing
* Default gateways
* DHCP relay

Although the Catalyst 3560 is capable of Layer 3 switching, R1 is intentionally used as the Layer 3 routing device in this project.

For a larger production network, a Layer 3 switch would often provide a more scalable routing architecture.

---

### Why centralize DHCP?

Centralized DHCP provides:

* Consistent address management
* Easier administration
* Centralized configuration
* A realistic enterprise service architecture

DHCP relay allows multiple VLANs to use the same DHCP infrastructure.

---

## 11. Current Network State

The baseline implementation has been successfully validated.

Verified functionality includes:

* VLAN 10 connectivity
* VLAN 20 connectivity
* VLAN 30 connectivity
* 802.1Q trunk operation between R1 and SW1
* 802.1Q trunk operation between SW1 and SW2
* Router-on-a-Stick
* DHCP address assignment
* DHCP relay
* Inter-VLAN routing
* Client-to-server connectivity
* R1-to-DHCP-server connectivity

This architecture represents the **baseline working network** before additional security and infrastructure features are introduced.
