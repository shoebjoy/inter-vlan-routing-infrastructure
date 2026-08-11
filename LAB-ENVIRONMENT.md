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

| Device      | Quantity | Model               |
| ----------- | -------: | ------------------- |
| Router      |        1 | Cisco 2911          |
| Switch      |        2 | Cisco Catalyst 2960 |
| DHCP Server |        1 | Generic Server      |
| End Devices |        6 | Generic PC          |

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
* Basic Network Validation
* Cisco IOS Configuration and Verification

---

## Project Files

| File                          | Description                                     |
| ----------------------------- | ----------------------------------------------- |
| topology.drawio               | Editable Draw.io topology                       |
| network-topology.png          | Exported topology diagram                       |
| vlan-based-office-network.pkt | Cisco Packet Tracer project                     |
| architecture.md               | Network architecture and design                 |
| implementation.md             | Device configuration and implementation process |
| ip-addressing.md              | IP addressing plan                              |
| troubleshooting.md            | Troubleshooting procedures and findings         |
| validation.md                 | Network validation and test results             |

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

Client-side IP configuration was verified using:

```text
ipconfig
```

Connectivity was validated using ICMP ping tests between:

* Clients and their default gateways
* Different VLANs
* Departmental VLANs and the DHCP server network

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

---

## Compatibility

This project has been tested using:

* Cisco Packet Tracer 9.0.1 (64-bit)

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
