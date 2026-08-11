# Lab Environment

This document describes the software, hardware, and testing environment used to build and validate this project.

---

## Host System

| Component        | Details                 |
| ---------------- | ----------------------- |
| Operating System | Windows 11 Pro (64-bit) |
| Architecture     | x64                     |
| Host Machine     | Lenovo Legion 9i        |

---

## Network Simulation Software

| Software            | Version        |
| ------------------- | -------------- |
| Cisco Packet Tracer | 9.0.1 (64-bit) |

---

## Network Devices

| Device      | Quantity | Model                    |
| ----------- | -------: | ------------------------ |
| Router      |        1 | Cisco 2911               |
| Switch      |        1 | Cisco Catalyst 3560-24PS |
| Switch      |        1 | Cisco Catalyst 2960      |
| DHCP Server |        1 | Generic Server           |
| End Devices |        6 | Generic PC               |

### Device Identification

| Device      | Model                    |
| ----------- | ------------------------ |
| R1          | Cisco 2911               |
| SW1         | Cisco Catalyst 3560-24PS |
| SW2         | Cisco Catalyst 2960      |
| DHCP Server | Generic Server           |

---

## Technologies Implemented

* VLAN Segmentation
* IEEE 802.1Q Trunking
* Access Ports
* Router-on-a-Stick (ROAS)
* Inter-VLAN Routing
* Centralized DHCP
* DHCP Relay (`ip helper-address`)
* Dedicated Server Network
* IP Addressing and Subnetting
* ICMP Connectivity Testing
* Basic Network Validation
* Cisco IOS Configuration and Verification
* Layer 2 and Layer 3 Troubleshooting

---

## Project Files

| File                                | Description                                     |
| ----------------------------------- | ----------------------------------------------- |
| `topology/topology.drawio`          | Editable Draw.io topology source                |
| `topology/network-topology.png`     | Exported network topology diagram               |
| `lab/vlan-based-office-network.pkt` | Cisco Packet Tracer project                     |
| `configs/R1.cfg`                    | R1 saved configuration                          |
| `configs/SW1.cfg`                   | SW1 saved configuration                         |
| `configs/SW2.cfg`                   | SW2 saved configuration                         |
| `docs/architecture.md`              | Network architecture and design                 |
| `docs/implementation.md`            | Device configuration and implementation process |
| `docs/ip-addressing.md`             | IP addressing plan                              |
| `docs/troubleshooting.md`           | Troubleshooting procedures and findings         |
| `docs/validation.md`                | Network validation and test results             |

---

## Network Addressing

The lab uses separate networks for departmental VLANs and centralized infrastructure services.

| Network            | Purpose                  |
| ------------------ | ------------------------ |
| `192.168.10.0/24`  | HR VLAN                  |
| `192.168.20.0/24`  | Sales VLAN               |
| `192.168.30.0/24`  | IT VLAN                  |
| `192.168.100.0/24` | Dedicated Server Network |

The DHCP server uses:

```text
IP Address:      192.168.100.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.100.1
```

The router provides the default gateway for each network:

```text
VLAN 10 → 192.168.10.1
VLAN 20 → 192.168.20.1
VLAN 30 → 192.168.30.1
Server  → 192.168.100.1
```

---

## Verification Commands

The following Cisco IOS commands were used during implementation and validation:

```bash
show vlan brief
show interfaces trunk
show interfaces gigabitEthernet0/1 switchport
show ip interface brief
show running-config
ping
```

Additional router verification was performed using:

```bash
show running-config | section interface GigabitEthernet0/0
```

Client-side IP configuration was verified using:

```text
ipconfig
```

Connectivity was validated using ICMP ping tests between:

* Clients and their default gateways
* Clients across different VLANs
* Departmental VLANs and the DHCP server network
* R1 and the DHCP server

---

## DHCP Environment

The lab uses a centralized DHCP architecture.

The DHCP server is located on the dedicated server network:

```text
DHCP Server
192.168.100.10/24
```

The departmental VLANs do not contain individual DHCP servers.

Instead, R1 forwards DHCP requests using DHCP relay:

```cisco
interface GigabitEthernet0/0.10
 ip helper-address 192.168.100.10

interface GigabitEthernet0/0.20
 ip helper-address 192.168.100.10

interface GigabitEthernet0/0.30
 ip helper-address 192.168.100.10
```

This allows clients in VLAN 10, VLAN 20, and VLAN 30 to obtain their IP configuration from the centralized DHCP server.

---

## Validation Performed

The network implementation was validated using:

* VLAN Verification
* Access-Port Verification
* 802.1Q Trunk Verification
* Router Interface Verification
* Router Subinterface Verification
* DHCP Address Assignment
* DHCP Relay Verification
* Default Gateway Connectivity
* Inter-VLAN Connectivity Testing
* DHCP Server Connectivity
* Running Configuration Verification
* End-to-End Connectivity Testing

### Client Validation

DHCP address assignment was verified on departmental clients using:

```text
ipconfig
```

The expected addressing structure was:

```text
HR Clients
192.168.10.0/24
Gateway: 192.168.10.1

Sales Clients
192.168.20.0/24
Gateway: 192.168.20.1

IT Clients
192.168.30.0/24
Gateway: 192.168.30.1
```

### Connectivity Validation

ICMP connectivity was tested between:

```text
Client → Default Gateway
Client → Other VLAN
Client → DHCP Server
R1 → DHCP Server
```

These tests confirmed Layer 3 forwarding between the departmental VLANs and connectivity to the centralized server network.

---

## Compatibility

This project has been tested using:

* Cisco Packet Tracer 9.0.1 (64-bit)
* Cisco 2911 router
* Cisco Catalyst 3560-24PS switch
* Cisco Catalyst 2960 switch

Other Packet Tracer versions may produce minor interface, device, or CLI differences, but the underlying networking concepts and configuration approach remain the same.

---

## Last Tested

| Item      | Value            |
| --------- | ---------------- |
| Tested By | Shoeb Mahmud Joy |
| Date      | 2026-08-11       |

---

## Notes

This lab was developed as a small enterprise campus networking project to demonstrate Cisco Layer 2 switching, VLAN segmentation, IEEE 802.1Q trunking, Router-on-a-Stick (ROAS), Inter-VLAN Routing, centralized DHCP deployment, DHCP relay, dedicated server networking, and systematic network validation using Cisco IOS commands.

The lab intentionally uses two different Cisco switch platforms:

* **SW1 — Cisco Catalyst 3560-24PS**
* **SW2 — Cisco Catalyst 2960**

The routing function is intentionally handled by the Cisco 2911 router using Router-on-a-Stick rather than by the multilayer switching capabilities of the Catalyst 3560. This keeps the implementation focused on VLAN trunking, router subinterfaces, Inter-VLAN Routing, and centralized DHCP relay.
