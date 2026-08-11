# Network Troubleshooting

## 1. Overview

This document records the troubleshooting process performed during the implementation of the Enterprise LAN network.

The objective is to document:

* The problems encountered during implementation
* The symptoms observed
* The troubleshooting approach
* The commands used to identify problems
* The corrective actions
* The final results

The troubleshooting process focused primarily on:

1. Trunk interface status
2. Router interface status
3. DHCP server connectivity
4. DHCP relay configuration
5. Inter-VLAN connectivity
6. Initial ICMP packet loss
7. End-device IP configuration

---

# 2. Troubleshooting Methodology

A structured troubleshooting approach was used instead of changing configurations randomly.

The general process was:

```text
1. Identify the symptom
        ↓
2. Check physical/link status
        ↓
3. Check Layer 2 configuration
        ↓
4. Check Layer 3 configuration
        ↓
5. Check addressing
        ↓
6. Check DHCP / services
        ↓
7. Test connectivity
        ↓
8. Verify the correction
```

The primary troubleshooting commands used were:

```cisco
show ip interface brief
show interfaces gigabitEthernet0/1 switchport
show vlan brief
show running-config
ping <destination-ip>
```

---

# 3. Issue: SW1 Trunk Initially Down

## Symptom

The SW1 interface connected toward the router was configured as a trunk, but the operational state initially showed:

```text
Administrative Mode: trunk
Operational Mode: down
```

The interface configuration itself was present:

```cisco
interface gigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

However, the operational trunk was not yet established.

---

## Investigation

The following command was used:

```cisco
show interfaces gigabitEthernet0/1 switchport
```

The important output was:

```text
Administrative Mode: trunk
Operational Mode: down
```

This indicated that the interface had been configured administratively as a trunk, but the physical/link state was not yet operational.

---

## Root Cause

The physical link between the devices was not yet up.

The switch subsequently reported:

```text
%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
```

---

## Resolution

After the connected interface became operational, the trunk was checked again:

```cisco
do show interfaces gigabitEthernet0/1 switchport
```

The final state showed:

```text
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Trunking VLANs Enabled: 10,20,30
```

---

## Result

The SW1 trunk became fully operational.

**Status: RESOLVED**

---

# 4. Issue: SW2 Trunk Verification

## Symptom

Before moving forward with DHCP and end-device configuration, the SW2 uplink needed to be verified.

---

## Investigation

The following command was used:

```cisco
do show interfaces gigabitEthernet0/1 switchport
```

The final output showed:

```text
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Trunking VLANs Enabled: 10,20,30
```

---

## Resolution

No configuration change was required.

The SW2 uplink was already operating correctly as an 802.1Q trunk.

---

## Result

SW2 trunk validation passed.

**Status: VERIFIED**

---

# 5. Issue: Router Interface Initially Not Enabled

## Symptom

The router interface connected to the server network had an IP address configured but had not yet been enabled.

The intended configuration was:

```text
IP Address: 192.168.100.1
Subnet Mask: 255.255.255.0
```

---

## Investigation

The interface was checked using:

```cisco
do show ip interface brief
```

An interface that is configured but not enabled can appear as:

```text
administratively down
```

---

## Resolution

The interface was enabled:

```cisco
interface gigabitEthernet0/1
 no shutdown
```

The interface subsequently became:

```text
GigabitEthernet0/1
192.168.100.1
up
up
```

---

## Result

The router successfully established Layer 3 connectivity with the server network.

**Status: RESOLVED**

---

# 6. Issue: Verifying DHCP Server Connectivity

## Symptom

Before configuring DHCP relay, connectivity between R1 and the DHCP server needed to be confirmed.

The DHCP server was configured as:

```text
IP Address:      192.168.100.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.100.1
```

R1 was configured as:

```text
192.168.100.1/24
```

---

## Investigation

From R1:

```cisco
do ping 192.168.100.10
```

Initial result:

```text
Success rate is 80 percent (4/5)
```

The important observation was that the majority of packets successfully reached the server.

---

## Analysis

The first packet timeout was consistent with initial Layer 2 address resolution.

Before normal ICMP communication can occur, the router may need to resolve the destination's MAC address using ARP.

Therefore, an initial timeout does not necessarily indicate a configuration failure when subsequent packets succeed.

---

## Resolution

No configuration change was required.

The connectivity test was repeated and communication remained successful.

---

## Result

R1 and the DHCP server were confirmed to be reachable.

**Status: VERIFIED**

---

# 7. Issue: DHCP Clients Needed Relay Configuration

## Symptom

The DHCP server was located in:

```text
192.168.100.0/24
```

while the clients were located in:

```text
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
```

DHCP discovery messages from clients are initially broadcast-based.

A router does not normally forward Layer 2 broadcasts between different networks.

Therefore, the clients required DHCP relay.

---

## Investigation

The router subinterfaces were checked:

```cisco
show running-config | section interface GigabitEthernet0/0
```

The VLAN gateway configuration existed, but DHCP relay needed to be configured.

---

## Resolution

The DHCP server address was configured as the helper address on each VLAN subinterface.

### VLAN 10

```cisco
interface gigabitEthernet0/0.10
 ip helper-address 192.168.100.10
```

### VLAN 20

```cisco
interface gigabitEthernet0/0.20
 ip helper-address 192.168.100.10
```

### VLAN 30

```cisco
interface gigabitEthernet0/0.30
 ip helper-address 192.168.100.10
```

---

## Verification

The configuration was verified using:

```cisco
do show running-config | section interface GigabitEthernet0/0
```

The final configuration contained:

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

## Result

The three VLANs could use the centralized DHCP server.

**Status: RESOLVED**

---

# 8. Issue: Verifying Client IP Configuration

## Symptom

After DHCP configuration, the client devices needed to be checked to ensure they received addresses from the correct DHCP pool.

---

## Investigation

The `ipconfig` command was used on the client PCs.

### HR-PC1

```text
IPv4 Address:      192.168.10.10
Subnet Mask:       255.255.255.0
Default Gateway:   192.168.10.1
```

### SA-PC1

```text
IPv4 Address:      192.168.20.10
Subnet Mask:       255.255.255.0
Default Gateway:   192.168.20.1
```

### IT-PC1

```text
IPv4 Address:      192.168.30.10
Subnet Mask:       255.255.255.0
Default Gateway:   192.168.30.1
```

---

## Analysis

Each client received an address from the correct VLAN subnet.

This confirmed that:

```text
VLAN 10 → 192.168.10.0/24
VLAN 20 → 192.168.20.0/24
VLAN 30 → 192.168.30.0/24
```

were correctly mapped to their respective DHCP pools.

---

## Result

Client addressing was correct.

**Status: VERIFIED**

---

# 9. Issue: Verifying VLAN Gateway Connectivity

Before testing inter-VLAN routing, each PC was tested against its local default gateway.

### HR

```text
ping 192.168.10.1
```

Result:

```text
4/4 replies
0% packet loss
```

### Sales

```text
ping 192.168.20.1
```

Result:

```text
4/4 replies
0% packet loss
```

### IT

```text
ping 192.168.30.1
```

Result:

```text
4/4 replies
0% packet loss
```

---

## Analysis

Successful gateway communication confirmed that:

* The PC was correctly addressed.
* The access port was associated with the correct VLAN.
* The VLAN existed.
* The trunk path was functional.
* The router subinterface was operational.

---

## Result

All departmental VLAN gateways were reachable.

**Status: VERIFIED**

---

# 10. Issue: Verifying Inter-VLAN Routing

## Symptom

After confirming local gateway connectivity, communication between different VLANs needed to be tested.

---

## Investigation

Tests were performed between departmental clients.

### HR → Sales

```text
ping 192.168.20.10
```

Result:

```text
4/4 replies
0% packet loss
```

### HR → IT

```text
ping 192.168.30.10
```

Result:

```text
4/4 replies
0% packet loss
```

### Sales → IT

```text
ping 192.168.30.10
```

Result:

```text
4/4 replies
0% packet loss
```

---

## Analysis

Successful communication between different VLANs confirmed that R1 was correctly routing between:

```text
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
```

---

## Result

Inter-VLAN routing was functioning correctly.

**Status: VERIFIED**

---

# 11. Issue: Initial Ping Timeout to DHCP Server

## Symptom

When HR-PC1 first tested connectivity to the DHCP server:

```text
ping 192.168.100.10
```

the result was:

```text
Request timed out.
Reply from 192.168.100.10
Reply from 192.168.100.10
Reply from 192.168.100.10
```

Result:

```text
Packets: Sent = 4
Received = 3
Lost = 1
```

---

## Analysis

The first packet timeout was not treated as a network failure because the subsequent packets were successful.

This behavior is commonly associated with initial ARP resolution.

The host/router first needs to discover the Layer 2 destination information before normal packet forwarding can occur.

Once the ARP information is available, subsequent ICMP packets can succeed.

---

## Resolution

No configuration change was required.

The connectivity remained functional after the initial packet.

---

## Result

The network was considered operational.

**Status: NORMAL INITIAL BEHAVIOR**

---

# 12. Issue: Packet Tracer PC Command Prompt Has No `clear` Command

## Symptom

While testing connectivity from Packet Tracer PCs, attempts were made to clear the terminal using:

```text
clear
```

and:

```text
clean
```

Packet Tracer responded:

```text
Invalid Command.
```

---

## Cause

The Packet Tracer PC command prompt is not a full Windows Command Prompt or Linux shell.

It implements only a limited set of commands.

Therefore, commands such as:

```text
clear
clean
```

are not available.

---

## Resolution

There is no need to clear the command history for network validation.

A new command can simply be entered at:

```text
C:\>
```

If a visually clean terminal is required, the Packet Tracer PC terminal can be closed and reopened.

---

## Result

No network problem existed.

**Status: NOT A NETWORK FAILURE**

---

# 13. Final Troubleshooting Verification

After resolving and verifying the issues above, the following areas were confirmed:

```text
Physical Links
      ↓
Switching
      ↓
VLANs
      ↓
Trunks
      ↓
Router Subinterfaces
      ↓
IP Addressing
      ↓
DHCP
      ↓
DHCP Relay
      ↓
Inter-VLAN Routing
      ↓
Server Connectivity
      ↓
End-to-End Connectivity
```

All critical components were functioning as expected.

---

# 14. Troubleshooting Command Reference

The following commands were the primary troubleshooting tools used in this project.

## Check Interface Status

```cisco
show ip interface brief
```

Used to identify:

* `up/up`
* `down/down`
* `administratively down`

---

## Check Switchport and Trunk Status

```cisco
show interfaces gigabitEthernet0/1 switchport
```

Used to verify:

* Administrative mode
* Operational mode
* Encapsulation
* Allowed VLANs
* Native VLAN

---

## Check VLANs

```cisco
show vlan brief
```

Used to verify:

* VLAN existence
* VLAN names
* Access-port assignments

---

## Check Router Configuration

```cisco
show running-config
```

Used to inspect:

* Interfaces
* Subinterfaces
* IP addresses
* VLAN encapsulation
* DHCP relay configuration

---

## Test Layer 3 Connectivity

```cisco
ping <destination-ip>
```

Used to verify:

* Gateway connectivity
* Inter-VLAN connectivity
* Server connectivity
* End-to-end reachability

---

## Check Client Addressing

On Packet Tracer PCs:

```text
ipconfig
```

Used to verify:

* IPv4 address
* Subnet mask
* Default gateway
* DHCP assignment

---

# 15. Troubleshooting Lessons

The troubleshooting process reinforced several practical networking principles.

### 1. Administrative state and operational state are different

A port can be configured as a trunk but still show:

```text
Operational Mode: down
```

Configuration alone does not guarantee an operational link.

---

### 2. Always verify Layer 2 before blaming Layer 3

When inter-VLAN connectivity fails, check:

```text
Physical link
   ↓
VLAN
   ↓
Trunk
   ↓
Router subinterface
   ↓
IP configuration
   ↓
Routing
```

This avoids changing Layer 3 configuration when the actual problem is Layer 2.

---

### 3. DHCP across VLANs requires a relay

A centralized DHCP server cannot directly receive client broadcasts from another subnet.

The router therefore uses:

```cisco
ip helper-address 192.168.100.10
```

to relay DHCP requests.

---

### 4. The first ping is not always representative of a failure

An initial ICMP timeout can occur while ARP resolution is taking place.

Therefore, when troubleshooting ping:

* Don't immediately assume failure from one lost packet.
* Check whether subsequent packets succeed.
* Verify ARP and interface state when necessary.

---

### 5. Troubleshooting should be evidence-driven

Instead of repeatedly changing configuration, use commands and tests to narrow down the problem.

The general principle used throughout this project was:

```text
Observe
   ↓
Hypothesize
   ↓
Test
   ↓
Fix
   ↓
Verify
```

---

# 16. Final Status

All major implementation issues encountered during the lab were either resolved or verified as normal Packet Tracer behavior.

The final network successfully supports:

* VLAN segmentation
* 802.1Q trunking
* Router-on-a-Stick
* DHCP
* DHCP relay
* Inter-VLAN routing
* Server connectivity
* End-to-end communication

**Overall Troubleshooting Status: RESOLVED / VERIFIED**
