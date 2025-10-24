# Setup Guide - Multiswitch Layer Network Configuration

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Hardware Requirements](#hardware-requirements)
3. [Step-by-Step Setup](#step-by-step-setup)
4. [PC Configuration](#pc-configuration)
5. [Cloud Configuration](#cloud-configuration)
6. [Testing and Verification](#testing-and-verification)
7. [Troubleshooting](#troubleshooting)

## Prerequisites

### Software
- Cisco Packet Tracer (Version 7.3 or higher recommended)
- Basic knowledge of Cisco IOS commands
- Understanding of VLANs and trunking concepts

### Skills Required
- Cisco CLI navigation
- IP addressing and subnetting
- Basic routing concepts

## Hardware Requirements

### Devices Needed in Packet Tracer

| Device Type | Model | Quantity | Purpose |
|------------|-------|----------|---------|
| Multilayer Switch | Cisco 3560 | 1 | Core routing and inter-VLAN routing |
| Access Switch | Cisco 2960 | 3 | Access layer connectivity |
| Router | Cisco 2911 | 1 | External connectivity |
| PC | Generic PC | 6 | End devices |
| Cloud | Cloud-PT | 1 | External network simulation |

## Step-by-Step Setup

### Phase 1: Physical Topology Setup

#### 1.1 Add Devices to Workspace
1. Open Cisco Packet Tracer
2. Add the following devices from the device selection panel:
   - 1x 3560 Multilayer Switch (or 3560-24PS)
   - 3x 2960 Switches (or 2960-24TT)
   - 1x 2911 Router (or 1941)
   - 6x Generic PCs
   - 1x Cloud-PT

#### 1.2 Create Physical Connections
Use **Copper Straight-Through** cables for all connections:

**Multilayer Switch Connections:**
- FastEthernet0/1 → Switch1 FastEthernet0/24 (Trunk)
- FastEthernet0/2 → Switch2 FastEthernet0/24 (Trunk)
- FastEthernet0/3 → Switch3 FastEthernet0/24 (Trunk)
- GigabitEthernet0/1 → Router GigabitEthernet0/0

**Switch1 Connections:**
- FastEthernet0/1 → PC1 FastEthernet0
- FastEthernet0/2 → PC2 FastEthernet0

**Switch2 Connections:**
- FastEthernet0/1 → PC3 FastEthernet0
- FastEthernet0/2 → PC4 FastEthernet0

**Switch3 Connections:**
- FastEthernet0/1 → PC5 FastEthernet0
- FastEthernet0/2 → PC6 FastEthernet0

**Router Connection:**
- GigabitEthernet0/1 → Cloud Ethernet6

### Phase 2: Configure Multilayer Switch

1. Click on the Multilayer Switch
2. Go to CLI tab
3. Copy and paste the configuration from `configs/multilayer-switch-config.txt`
4. Alternatively, enter commands manually line by line
5. Verify configuration:
   ```
   show vlan brief
   show ip interface brief
   show interfaces trunk
   ```

### Phase 3: Configure Access Switches

#### Switch1
1. Click on Switch1
2. Go to CLI tab
3. Copy and paste configuration from `configs/switch1-config.txt`
4. Verify: `show vlan brief` and `show interfaces trunk`

#### Switch2
1. Click on Switch2
2. Go to CLI tab
3. Copy and paste configuration from `configs/switch2-config.txt`
4. Verify: `show vlan brief` and `show interfaces trunk`

#### Switch3
1. Click on Switch3
2. Go to CLI tab
3. Copy and paste configuration from `configs/switch3-config.txt`
4. Verify: `show vlan brief` and `show interfaces trunk`

### Phase 4: Configure Router

1. Click on the Router
2. Go to CLI tab
3. Copy and paste configuration from `configs/router-config.txt`
4. Verify:
   ```
   show ip interface brief
   show ip route
   show ip nat translations
   ```

## PC Configuration

Configure each PC with static IP addresses:

### PC1 (Sales - VLAN 10)
1. Click on PC1 → Desktop → IP Configuration
2. Set to **Static**
3. IP Address: `192.168.10.10`
4. Subnet Mask: `255.255.255.0`
5. Default Gateway: `192.168.10.1`
6. DNS Server: `8.8.8.8`

### PC2 (Marketing - VLAN 20)
1. Click on PC2 → Desktop → IP Configuration
2. IP Address: `192.168.20.10`
3. Subnet Mask: `255.255.255.0`
4. Default Gateway: `192.168.20.1`
5. DNS Server: `8.8.8.8`

### PC3 (Engineering - VLAN 30)
1. Click on PC3 → Desktop → IP Configuration
2. IP Address: `192.168.30.10`
3. Subnet Mask: `255.255.255.0`
4. Default Gateway: `192.168.30.1`
5. DNS Server: `8.8.8.8`

### PC4 (HR - VLAN 40)
1. Click on PC4 → Desktop → IP Configuration
2. IP Address: `192.168.40.10`
3. Subnet Mask: `255.255.255.0`
4. Default Gateway: `192.168.40.1`
5. DNS Server: `8.8.8.8`

### PC5 (Finance - VLAN 50)
1. Click on PC5 → Desktop → IP Configuration
2. IP Address: `192.168.50.10`
3. Subnet Mask: `255.255.255.0`
4. Default Gateway: `192.168.50.1`
5. DNS Server: `8.8.8.8`

### PC6 (IT - VLAN 60)
1. Click on PC6 → Desktop → IP Configuration
2. IP Address: `192.168.60.10`
3. Subnet Mask: `255.255.255.0`
4. Default Gateway: `192.168.60.1`
5. DNS Server: `8.8.8.8`

## Cloud Configuration

### Configure Cloud Connection

1. Click on Cloud-PT
2. Go to Config tab
3. Select Ethernet6 interface
4. Set IP Configuration:
   - IP Address: `203.0.113.2` (or as appropriate for your ISP simulation)
   - Subnet Mask: `255.255.255.0`
5. Alternatively, use DHCP if simulating real internet connection

## Testing and Verification

### Test 1: Intra-VLAN Communication
Test communication between devices in the same VLAN:

From PC1 (VLAN 10):
```
ping 192.168.10.1  (Gateway - should succeed)
```

### Test 2: Inter-VLAN Communication
Test communication between different VLANs:

From PC1 (VLAN 10):
```
ping 192.168.20.10  (PC2 in VLAN 20 - should succeed)
ping 192.168.30.10  (PC3 in VLAN 30 - should succeed)
ping 192.168.60.10  (PC6 in VLAN 60 - should succeed)
```

### Test 3: Router Connectivity
From any PC:
```
ping 10.0.0.1  (Router interface - should succeed)
```

### Test 4: External Connectivity (if Cloud configured)
From any PC:
```
ping 203.0.113.2  (Cloud - should succeed if NAT configured correctly)
ping 8.8.8.8      (Google DNS - should succeed with internet)
```

### Test 5: Trunk Verification
On Multilayer Switch:
```
show interfaces trunk
show vlan brief
```

Expected output should show:
- Trunk ports (Fa0/1, Fa0/2, Fa0/3) in trunk mode
- All VLANs active
- Correct VLAN assignments

## Troubleshooting

### Issue 1: PCs Cannot Ping Gateway

**Symptoms:** PC cannot reach its default gateway

**Solutions:**
1. Verify PC IP configuration is correct
2. Check VLAN assignment on access switch port:
   ```
   show vlan brief
   show interfaces status
   ```
3. Verify SVI is up on multilayer switch:
   ```
   show ip interface brief
   ```
4. Check if interface is shutdown:
   ```
   show interfaces vlan <vlan-id>
   ```

### Issue 2: Inter-VLAN Communication Fails

**Symptoms:** PCs in different VLANs cannot communicate

**Solutions:**
1. Verify IP routing is enabled on multilayer switch:
   ```
   show ip route
   ```
2. Check if all SVIs are up:
   ```
   show ip interface brief
   ```
3. Verify trunk links are operational:
   ```
   show interfaces trunk
   ```
4. Check if correct VLANs are allowed on trunk:
   ```
   show interfaces trunk
   ```

### Issue 3: Trunk Not Forming

**Symptoms:** show interfaces trunk shows empty or error

**Solutions:**
1. Verify both sides configured as trunk
2. Check encapsulation matches (dot1q)
3. Ensure native VLAN matches
4. Check physical connectivity
5. Manual configuration:
   ```
   interface <interface-id>
   switchport mode trunk
   switchport trunk encapsulation dot1q
   switchport trunk native vlan 99
   ```

### Issue 4: No Internet Access

**Symptoms:** Cannot reach external addresses

**Solutions:**
1. Verify router has route to internet:
   ```
   show ip route
   ```
2. Check NAT configuration:
   ```
   show ip nat translations
   show access-lists
   ```
3. Verify outside interface is up:
   ```
   show ip interface brief
   ```
4. Test from router:
   ```
   ping 8.8.8.8
   ```

### Issue 5: Port Security Violations

**Symptoms:** Ports go into err-disabled state

**Solutions:**
1. Check port status:
   ```
   show port-security interface <interface-id>
   ```
2. Clear violation and re-enable:
   ```
   configure terminal
   interface <interface-id>
   shutdown
   no shutdown
   ```

### Common Commands for Verification

```bash
# Check interface status
show ip interface brief

# Check VLAN configuration
show vlan brief

# Check trunk status
show interfaces trunk

# Check routing table
show ip route

# Check running configuration
show running-config

# Check MAC address table
show mac address-table

# Check port security
show port-security

# Check NAT translations
show ip nat translations
```

## Additional Notes

1. **Save Configurations:** Always save configurations after making changes:
   ```
   write memory
   # or
   copy running-config startup-config
   ```

2. **Backup:** Save your Packet Tracer file regularly

3. **Documentation:** Keep notes of any custom changes made to the configuration

4. **Security:** Change default passwords in production environments

5. **Scalability:** This design easily scales by adding more access switches and VLANs

## Next Steps

After completing the basic setup:
1. Implement spanning-tree configurations
2. Add redundancy with multiple links
3. Configure dynamic routing protocols (OSPF/EIGRP)
4. Implement ACLs for security
5. Add DHCP services
6. Configure QoS for traffic prioritization
