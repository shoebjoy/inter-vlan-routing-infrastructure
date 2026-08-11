# Network Implementation

## 1. Overview

This document describes the implementation procedure for the enterprise campus network.

The implementation consists of:

1. VLAN creation
2. Access-port assignment
3. 802.1Q trunk configuration
4. Router-on-a-Stick configuration
5. Server-network configuration
6. Centralized DHCP configuration
7. DHCP relay configuration
8. End-device DHCP configuration
9. Basic connectivity verification

The implementation uses the following VLANs:

| VLAN | Name  | Network         |
| ---: | ----- | --------------- |
|   10 | HR    | 192.168.10.0/24 |
|   20 | SALES | 192.168.20.0/24 |
|   30 | IT    | 192.168.30.0/24 |

The infrastructure/server network is:

```text
192.168.100.0/24
```

The DHCP server uses:

```text
192.168.100.10
```

---

# 2. SW1 Configuration

SW1 acts as the primary switch connecting the router to the downstream access switch.

## 2.1 Set Hostname

```cisco
enable
configure terminal
hostname SW1
```

---

## 2.2 Create VLANs

Create the three departmental VLANs:

```cisco
vlan 10
 name HR
exit

vlan 20
 name SALES
exit

vlan 30
 name IT
exit
```

---

## 2.3 Configure Router Trunk

The connection between SW1 and R1 is configured as an 802.1Q trunk.

```cisco
interface gigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

The trunk carries VLANs:

```text
10
20
30
```

---

## 2.4 Configure SW1-to-SW2 Trunk

The connection from SW1 to SW2 is also configured as an 802.1Q trunk.

```cisco
interface gigabitEthernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

---

## 2.5 Verify SW1 Trunks

The trunk status can be verified with:

```cisco
do show interfaces gigabitEthernet0/1 switchport
```

and:

```cisco
do show interfaces gigabitEthernet0/2 switchport
```

The expected operational mode is:

```text
Operational Mode: trunk
```

The expected allowed VLANs are:

```text
10,20,30
```

---

# 3. SW2 Configuration

SW2 acts as the access switch for the departmental client devices.

## 3.1 Set Hostname

```cisco
enable
configure terminal
hostname SW2
```

---

## 3.2 Create VLANs

```cisco
vlan 10
 name HR
exit

vlan 20
 name SALES
exit

vlan 30
 name IT
exit
```

---

## 3.3 Configure Uplink Trunk

The connection from SW2 to SW1 is configured as an 802.1Q trunk.

```cisco
interface gigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 no shutdown
exit
```

---

# 4. Configure SW2 Access Ports

The end devices are separated into departmental VLANs using access ports.

## 4.1 HR Ports

HR clients are connected to:

```text
FastEthernet0/1
FastEthernet0/2
```

Configure them as VLAN 10 access ports:

```cisco
interface range fastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
exit
```

---

## 4.2 SALES Ports

SALES clients are connected to:

```text
FastEthernet0/3
FastEthernet0/4
```

Configure them as VLAN 20 access ports:

```cisco
interface range fastEthernet0/3 - 4
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
exit
```

---

## 4.3 IT Ports

IT clients are connected to:

```text
FastEthernet0/5
FastEthernet0/6
```

Configure them as VLAN 30 access ports:

```cisco
interface range fastEthernet0/5 - 6
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
exit
```

---

## 4.4 Verify VLAN Assignment

Use:

```cisco
do show vlan brief
```

The expected result is:

```text
VLAN 10    HR       Fa0/1, Fa0/2
VLAN 20    SALES    Fa0/3, Fa0/4
VLAN 30    IT       Fa0/5, Fa0/6
```

---

# 5. R1 Router Configuration

R1 provides Layer 3 routing between the departmental VLANs.

The router uses Router-on-a-Stick.

## 5.1 Set Hostname

```cisco
enable
configure terminal
hostname R1
```

---

# 6. Configure R1-to-SW1 Interface

The physical interface connected to SW1 is:

```text
GigabitEthernet0/0
```

Enable the interface:

```cisco
interface gigabitEthernet0/0
 no shutdown
exit
```

The physical interface does not receive an IP address because the VLAN gateways are configured on subinterfaces.

---

# 7. Configure VLAN 10 Subinterface

The HR gateway is:

```text
192.168.10.1/24
```

Configure:

```cisco
interface gigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit
```

---

# 8. Configure VLAN 20 Subinterface

The SALES gateway is:

```text
192.168.20.1/24
```

Configure:

```cisco
interface gigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
```

---

# 9. Configure VLAN 30 Subinterface

The IT gateway is:

```text
192.168.30.1/24
```

Configure:

```cisco
interface gigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit
```

---

# 10. Configure Server Network Interface

R1 uses `GigabitEthernet0/1` to connect to the DHCP server network.

The server network is:

```text
192.168.100.0/24
```

Configure R1:

```cisco
interface gigabitEthernet0/1
 ip address 192.168.100.1 255.255.255.0
 no shutdown
exit
```

R1 now has:

```text
192.168.100.1/24
```

as the gateway for the server network.

---

# 11. Configure DHCP Server

The centralized DHCP server uses a static IP address.

Configure the server under:

```text
Desktop
→ IP Configuration
```

Use:

```text
IPv4 Address:    192.168.100.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.100.1
```

The DNS server field is not required for this lab and can remain:

```text
0.0.0.0
```

---

# 12. Configure DHCP Service

Navigate to:

```text
Services
→ DHCP
```

Ensure:

```text
DHCP Service: On
```

Create the following DHCP pools.

---

## 12.1 HR DHCP Pool

```text
Pool Name:           HR
Default Gateway:     192.168.10.1
DNS Server:          0.0.0.0
Start IP Address:    192.168.10.10
Subnet Mask:         255.255.255.0
Maximum Users:       50
```

Save the pool.

---

## 12.2 SALES DHCP Pool

```text
Pool Name:           SALES
Default Gateway:     192.168.20.1
DNS Server:          0.0.0.0
Start IP Address:    192.168.20.10
Subnet Mask:         255.255.255.0
Maximum Users:       50
```

Save the pool.

---

## 12.3 IT DHCP Pool

```text
Pool Name:           IT
Default Gateway:     192.168.30.1
DNS Server:          0.0.0.0
Start IP Address:    192.168.30.10
Subnet Mask:         255.255.255.0
Maximum Users:       50
```

Save the pool.

---

# 13. Configure DHCP Relay

DHCP clients are located in separate VLANs from the DHCP server.

Because DHCP discovery initially uses broadcast traffic, the router must relay DHCP requests toward the centralized server.

The DHCP server address is:

```text
192.168.100.10
```

---

## 13.1 VLAN 10 DHCP Relay

```cisco
interface gigabitEthernet0/0.10
 ip helper-address 192.168.100.10
exit
```

---

## 13.2 VLAN 20 DHCP Relay

```cisco
interface gigabitEthernet0/0.20
 ip helper-address 192.168.100.10
exit
```

---

## 13.3 VLAN 30 DHCP Relay

```cisco
interface gigabitEthernet0/0.30
 ip helper-address 192.168.100.10
exit
```

The resulting configuration is:

```text
VLAN 10 → R1 → DHCP Server
VLAN 20 → R1 → DHCP Server
VLAN 30 → R1 → DHCP Server
```

---

# 14. Configure Client PCs for DHCP

Each client PC is configured to obtain its IPv4 configuration automatically.

Navigate to:

```text
Desktop
→ IP Configuration
→ DHCP
```

This is applied to the client devices in:

```text
HR
SALES
IT
```

The clients should receive addresses from their corresponding DHCP pools.

Expected address ranges:

```text
HR:
192.168.10.10+
 
SALES:
192.168.20.10+

IT:
192.168.30.10+
```

The default gateways should correspond to their VLAN:

```text
HR    → 192.168.10.1
SALES → 192.168.20.1
IT    → 192.168.30.1
```

---

# 15. Verify R1 Interface Configuration

Run:

```cisco
do show ip interface brief
```

The expected interfaces are:

```text
GigabitEthernet0/0       up/up
GigabitEthernet0/0.10    192.168.10.1    up/up
GigabitEthernet0/0.20    192.168.20.1    up/up
GigabitEthernet0/0.30    192.168.30.1    up/up
GigabitEthernet0/1       192.168.100.1   up/up
```

---

# 16. Verify DHCP Relay Configuration

Run:

```cisco
do show running-config | section interface GigabitEthernet0/0
```

The VLAN subinterfaces should contain:

```text
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.100.10

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.100.10

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip helper-address 192.168.100.10
```

---

# 17. Save Device Configurations

After completing the configuration, save the running configuration on each Cisco device.

## R1

```cisco
copy running-config startup-config
```

## SW1

```cisco
copy running-config startup-config
```

## SW2

```cisco
copy running-config startup-config
```

When prompted for the destination filename, press **Enter** to accept the default.

---

# 18. Export Configurations for Version Control

The final configurations are exported as text files and stored in the Git repository.

Recommended structure:

```text
configs/
├── R1-running-config.txt
├── SW1-running-config.txt
└── SW2-running-config.txt
```

The configurations can be obtained with:

```cisco
show running-config
```

The output is then saved as the corresponding text file.

---

# 19. Implementation Result

At the completion of this implementation, the network provides:

```text
VLAN 10 ─┐
         │
VLAN 20 ─┼── R1 ─── DHCP Server
         │
VLAN 30 ─┘
```

The implementation provides:

* Departmental VLAN segmentation
* 802.1Q trunking
* Router-on-a-Stick inter-VLAN routing
* Dedicated server network
* Centralized DHCP
* DHCP relay
* Automatic client IP configuration
* Inter-VLAN communication
* Client-to-server connectivity

This configuration establishes the **working baseline** for the network.

Further security controls such as ACLs, DHCP Snooping, Dynamic ARP Inspection, Port Security, and BPDU Guard can be added in later implementation phases.
