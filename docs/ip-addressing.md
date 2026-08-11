# IP Addressing

## 1. IP Addressing Overview

The network uses separate IPv4 subnets for each department VLAN and a dedicated transit network between the router and DHCP server.

| Network        | VLAN | Subnet             | Default Gateway |
| -------------- | ---: | ------------------ | --------------- |
| HR             |   10 | `192.168.10.0/24`  | `192.168.10.1`  |
| Sales          |   20 | `192.168.20.0/24`  | `192.168.20.1`  |
| IT             |   30 | `192.168.30.0/24`  | `192.168.30.1`  |
| Server Network |    — | `192.168.100.0/24` | `192.168.100.1` |

---

## 2. VLAN Addressing

### VLAN 10 — HR

* **VLAN ID:** `10`
* **Network:** `192.168.10.0/24`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `192.168.10.1`
* **DHCP Server:** `192.168.100.10`
* **DHCP Pool:** `HR`

Example clients:

| Device | IP Address      |
| ------ | --------------- |
| HR-PC1 | `192.168.10.10` |
| HR-PC2 | DHCP            |

---

### VLAN 20 — Sales

* **VLAN ID:** `20`
* **Network:** `192.168.20.0/24`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `192.168.20.1`
* **DHCP Server:** `192.168.100.10`
* **DHCP Pool:** `SALES`

Example clients:

| Device | IP Address      |
| ------ | --------------- |
| SA-PC1 | `192.168.20.10` |
| SA-PC2 | DHCP            |

---

### VLAN 30 — IT

* **VLAN ID:** `30`
* **Network:** `192.168.30.0/24`
* **Subnet Mask:** `255.255.255.0`
* **Default Gateway:** `192.168.30.1`
* **DHCP Server:** `192.168.100.10`
* **DHCP Pool:** `IT`

Example clients:

| Device | IP Address      |
| ------ | --------------- |
| IT-PC1 | `192.168.30.10` |
| IT-PC2 | DHCP            |

---

## 3. Server Network

The DHCP server is located on a separate server network.

| Device   | Interface          | IP Address       | Subnet Mask     | Gateway         |
| -------- | ------------------ | ---------------- | --------------- | --------------- |
| DHCP-SRV | FastEthernet0      | `192.168.100.10` | `255.255.255.0` | `192.168.100.1` |
| R1       | GigabitEthernet0/1 | `192.168.100.1`  | `255.255.255.0` | —               |

The server uses a static IP address because infrastructure services such as DHCP should not depend on DHCP itself for their own addressing.

---

## 4. Router Subinterfaces

Router-on-a-Stick is used to provide inter-VLAN routing.

| Router Interface |           VLAN | IP Address         |
| ---------------- | -------------: | ------------------ |
| `G0/0.10`        |             10 | `192.168.10.1/24`  |
| `G0/0.20`        |             20 | `192.168.20.1/24`  |
| `G0/0.30`        |             30 | `192.168.30.1/24`  |
| `G0/1`           | Server Network | `192.168.100.1/24` |

Configuration concept:

```cisco
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

interface GigabitEthernet0/1
 ip address 192.168.100.1 255.255.255.0
 no shutdown
```

---

## 5. DHCP Address Allocation

The DHCP server provides addresses for the three departmental VLANs.

### HR DHCP Pool

```text
Pool Name:       HR
Network:         192.168.10.0/24
Gateway:         192.168.10.1
Starting Address: 192.168.10.10
Maximum Users:   50
```

### Sales DHCP Pool

```text
Pool Name:       SALES
Network:         192.168.20.0/24
Gateway:         192.168.20.1
Starting Address: 192.168.20.10
Maximum Users:   50
```

### IT DHCP Pool

```text
Pool Name:       IT
Network:         192.168.30.0/24
Gateway:         192.168.30.1
Starting Address: 192.168.30.10
Maximum Users:   50
```

---

## 6. DHCP Relay

Because the DHCP server is located on a different subnet from the client VLANs, DHCP broadcast packets cannot reach it directly.

The router acts as a DHCP relay using:

```cisco
ip helper-address 192.168.100.10
```

The helper address is configured on each VLAN subinterface:

```text
VLAN 10 → 192.168.100.10
VLAN 20 → 192.168.100.10
VLAN 30 → 192.168.100.10
```

This allows clients in all three VLANs to obtain their IP configuration from the centralized DHCP server.

---

## 7. Addressing Design

The addressing scheme follows a simple departmental structure:

```text
192.168.10.0/24 → HR
192.168.20.0/24 → Sales
192.168.30.0/24 → IT
192.168.100.0/24 → Servers
```

The third octet corresponds to the VLAN identifier for the departmental networks.

This makes the addressing scheme easier to understand, troubleshoot, document, and expand.

---

## 8. Address Allocation Policy

The network uses the following general allocation policy:

* `.1` → Default gateway
* `.10` onward → DHCP client range
* Server infrastructure → Static addressing
* Departmental end devices → DHCP
* Separate subnet → Server network

Example:

```text
HR
192.168.10.1     → Gateway
192.168.10.10+   → DHCP clients

Sales
192.168.20.1     → Gateway
192.168.20.10+   → DHCP clients

IT
192.168.30.1     → Gateway
192.168.30.10+   → DHCP clients

Servers
192.168.100.1    → Router
192.168.100.10   → DHCP Server
```

---

## 9. Connectivity Validation

The addressing design was validated using ICMP ping tests.

### Same-VLAN Gateway Tests

HR:

```text
HR-PC1 → 192.168.10.1
Result: 0% packet loss
```

Sales:

```text
SA-PC1 → 192.168.20.1
Result: 0% packet loss
```

IT:

```text
IT-PC1 → 192.168.30.1
Result: 0% packet loss
```

### Inter-VLAN Tests

Successful communication was verified between:

```text
HR → Sales
HR → IT
Sales → IT
```

This confirms that the router is correctly performing inter-VLAN routing.

### Server Connectivity Tests

The departmental clients were also tested against:

```text
DHCP-SRV → 192.168.100.10
```

Successful responses confirmed connectivity between the departmental VLANs and the dedicated server network.

---

## 10. Final IP Addressing Summary

| Device/Network | VLAN | IP/Subnet           | Purpose        |
| -------------- | ---: | ------------------- | -------------- |
| R1 G0/0.10     |   10 | `192.168.10.1/24`   | HR Gateway     |
| R1 G0/0.20     |   20 | `192.168.20.1/24`   | Sales Gateway  |
| R1 G0/0.30     |   30 | `192.168.30.1/24`   | IT Gateway     |
| R1 G0/1        |    — | `192.168.100.1/24`  | Server Gateway |
| DHCP-SRV       |    — | `192.168.100.10/24` | DHCP Server    |
| HR-PC1         |   10 | `192.168.10.10`     | HR Client      |
| HR-PC2         |   10 | DHCP                | HR Client      |
| SA-PC1         |   20 | `192.168.20.10`     | Sales Client   |
| SA-PC2         |   20 | DHCP                | Sales Client   |
| IT-PC1         |   30 | `192.168.30.10`     | IT Client      |
| IT-PC2         |   30 | DHCP                | IT Client      |

---

## 11. Network Addressing Diagram

```text
                         DHCP-SRV
                      192.168.100.10
                             |
                             |
                       192.168.100.0/24
                             |
                         R1 G0/1
                      192.168.100.1
                             |
                             |
                         R1 G0/0
                             |
                    Router-on-a-Stick
                  _________|_________
                 /         |         \
                /          |          \
          VLAN 10       VLAN 20      VLAN 30
        192.168.10.0  192.168.20.0  192.168.30.0
             |             |             |
         HR Gateway    Sales Gateway   IT Gateway
         .10.1          .20.1          .30.1
```

---

## 12. Design Notes

This addressing design provides:

* Clear separation between departments
* Dedicated server subnet
* Centralized DHCP management
* Router-based inter-VLAN routing
* Predictable default gateways
* Simple troubleshooting
* Easy future expansion

The design can later be extended with additional VLANs, dedicated management networks, network security controls, DNS services, monitoring infrastructure, and redundant gateways.
