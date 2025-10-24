# VLAN Configuration Guide

## Overview
This guide provides detailed information about VLAN configuration in the multiswitch layer network, including creation, assignment, and best practices.

## What are VLANs?

**VLAN (Virtual Local Area Network)** is a logical grouping of devices that can communicate as if they were on the same physical network, regardless of their physical location. VLANs provide:

- **Security:** Segregation of sensitive data
- **Performance:** Reduced broadcast domains
- **Management:** Easier network administration
- **Flexibility:** Logical grouping regardless of location

## VLAN Design

### VLAN Table

| VLAN ID | Name        | Subnet           | Gateway       | Purpose                  | Devices        |
|---------|-------------|------------------|---------------|--------------------------|----------------|
| 10      | Sales       | 192.168.10.0/24  | 192.168.10.1  | Sales Department         | PC1            |
| 20      | Marketing   | 192.168.20.0/24  | 192.168.20.1  | Marketing Department     | PC2            |
| 30      | Engineering | 192.168.30.0/24  | 192.168.30.1  | Engineering Department   | PC3            |
| 40      | HR          | 192.168.40.0/24  | 192.168.40.1  | Human Resources          | PC4            |
| 50      | Finance     | 192.168.50.0/24  | 192.168.50.1  | Finance Department       | PC5            |
| 60      | IT          | 192.168.60.0/24  | 192.168.60.1  | IT Department            | PC6            |
| 99      | Management  | 192.168.99.0/24  | 192.168.99.1  | Network Management       | All switches   |

### VLAN 1 (Default VLAN)
- **Not Used:** For security reasons, VLAN 1 is not used for any production traffic
- **Best Practice:** Disable unused ports and avoid using VLAN 1

## Creating VLANs

### On Multilayer Switch

```cisco
enable
configure terminal

! Create VLANs
vlan 10
 name Sales
 exit

vlan 20
 name Marketing
 exit

vlan 30
 name Engineering
 exit

vlan 40
 name HR
 exit

vlan 50
 name Finance
 exit

vlan 60
 name IT
 exit

vlan 99
 name Management
 exit
```

### On Access Switches

Each access switch only needs VLANs it will use:

**Switch1 (Sales & Marketing):**
```cisco
vlan 10
 name Sales
 exit
vlan 20
 name Marketing
 exit
vlan 99
 name Management
 exit
```

**Switch2 (Engineering & HR):**
```cisco
vlan 30
 name Engineering
 exit
vlan 40
 name HR
 exit
vlan 99
 name Management
 exit
```

**Switch3 (Finance & IT):**
```cisco
vlan 50
 name Finance
 exit
vlan 60
 name IT
 exit
vlan 99
 name Management
 exit
```

## Assigning Ports to VLANs

### Access Port Configuration

Access ports are connected to end devices (PCs, printers, etc.)

**Example: Assign port to VLAN 10**
```cisco
interface FastEthernet0/1
 description PC1 - Sales Department
 switchport mode access
 switchport access vlan 10
 no shutdown
 exit
```

**Key Points:**
- `switchport mode access` - Sets port as access port
- `switchport access vlan 10` - Assigns port to VLAN 10
- Only one VLAN per access port

## Inter-VLAN Routing

### Switch Virtual Interfaces (SVIs)

SVIs are virtual interfaces on the multilayer switch that serve as gateways for VLANs.

**Creating SVIs:**
```cisco
! Enable IP routing first
ip routing

! Create SVI for VLAN 10
interface vlan 10
 description Gateway for Sales VLAN
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

! Create SVI for VLAN 20
interface vlan 20
 description Gateway for Marketing VLAN
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit

! Repeat for all VLANs...
```

### Routing Between VLANs

With SVIs configured and `ip routing` enabled:
1. PC in VLAN 10 sends packet to PC in VLAN 20
2. Packet goes to default gateway (192.168.10.1 - SVI on multilayer switch)
3. Multilayer switch routes packet to VLAN 20's SVI (192.168.20.1)
4. Packet forwarded to destination PC through appropriate trunk and access switch

## Trunk Configuration

### What is a Trunk?

A trunk link carries traffic for multiple VLANs between switches using VLAN tagging (802.1Q).

### Configuring Trunk Ports

**On Multilayer Switch:**
```cisco
interface FastEthernet0/1
 description Trunk to Switch1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit
```

**On Access Switch:**
```cisco
interface FastEthernet0/24
 description Trunk to Multilayer Switch
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit
```

### Trunk Configuration Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| Encapsulation | Tagging protocol | dot1q (802.1Q) |
| Mode | Port mode | trunk |
| Native VLAN | Untagged VLAN | 99 |
| Allowed VLANs | VLANs permitted on trunk | 10,20,99 |

### Native VLAN

- **Purpose:** Carries untagged traffic on trunk
- **Configuration:** Must match on both ends
- **Best Practice:** Use dedicated VLAN (not VLAN 1)
- **Our Setup:** VLAN 99 (Management)

## VLAN Best Practices

### 1. VLAN Naming
- Use descriptive names (Sales, Marketing, not VLAN10, VLAN20)
- Maintain consistent naming across all switches
- Document VLAN purposes

### 2. VLAN Numbering
- Reserve VLANs 1-99 for special purposes
- Use 100-999 for user VLANs in larger networks
- Our setup uses 10, 20, 30, 40, 50, 60 for simplicity

### 3. Security
- Disable unused ports: `shutdown`
- Don't use VLAN 1 for production
- Implement port security on access ports
- Use separate VLAN for management

### 4. Native VLAN
- Always configure native VLAN explicitly
- Use dedicated VLAN (not VLAN 1)
- Ensure it matches on both trunk ends

### 5. Allowed VLANs on Trunks
- Only allow necessary VLANs
- Reduces broadcast domain
- Improves security

### 6. Documentation
- Maintain VLAN database
- Document all VLAN assignments
- Keep configuration backups

## Verification Commands

### Check VLAN Database
```cisco
show vlan brief
show vlan id 10
show vlan name Sales
```

### Check VLAN Interfaces
```cisco
show ip interface brief
show interfaces vlan 10
```

### Check Trunk Status
```cisco
show interfaces trunk
show interfaces FastEthernet0/1 switchport
```

### Check VLAN Assignment
```cisco
show vlan
show interfaces status
```

### Check Routing Table
```cisco
show ip route
show ip route connected
```

## Troubleshooting VLANs

### Issue 1: Device Cannot Communicate Within VLAN

**Check:**
1. VLAN exists on switch: `show vlan brief`
2. Port assigned to correct VLAN: `show vlan id <vlan-id>`
3. Port is not shutdown: `show interfaces status`
4. PC has correct IP configuration

**Fix:**
```cisco
configure terminal
interface <interface-id>
 switchport mode access
 switchport access vlan <vlan-id>
 no shutdown
 exit
```

### Issue 2: Cannot Communicate Between VLANs

**Check:**
1. IP routing enabled: `show ip route`
2. SVIs configured and up: `show ip interface brief`
3. Trunk carrying VLANs: `show interfaces trunk`
4. Default gateway configured on PCs

**Fix:**
```cisco
! On multilayer switch
configure terminal
ip routing

interface vlan <vlan-id>
 ip address <gateway-ip> <subnet-mask>
 no shutdown
 exit
```

### Issue 3: Trunk Not Working

**Check:**
1. Both sides configured as trunk
2. Encapsulation matches (dot1q)
3. Native VLAN matches
4. VLANs allowed on trunk

**Fix:**
```cisco
! On both switches
configure terminal
interface <interface-id>
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan <vlan-list>
 no shutdown
 exit
```

### Issue 4: VLAN Not Showing

**Check:**
1. VLAN created: `show vlan brief`
2. VLAN not shutdown

**Fix:**
```cisco
configure terminal
vlan <vlan-id>
 name <vlan-name>
 no shutdown
 exit
```

## Advanced VLAN Features

### 1. VLAN Pruning
Restricts unnecessary VLAN traffic from traversing trunk links.

```cisco
! Manually specify allowed VLANs
interface FastEthernet0/1
 switchport trunk allowed vlan 10,20,99
 exit
```

### 2. Voice VLANs
Separate voice traffic from data traffic.

```cisco
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 100
 exit
```

### 3. Private VLANs
Provide isolation between ports in same VLAN.

### 4. VLAN Access Control Lists (VACLs)
Filter traffic within a VLAN.

## Scaling Considerations

### Adding New VLANs
1. Plan IP addressing scheme
2. Create VLAN on all necessary switches
3. Configure SVI on multilayer switch
4. Update trunk allowed VLANs list
5. Assign access ports to new VLAN
6. Test connectivity

### Adding New Devices
1. Determine VLAN assignment
2. Configure access port on switch
3. Configure IP on device
4. Test connectivity

### Adding New Switches
1. Configure VLANs on new switch
2. Configure trunk to multilayer switch
3. Configure access ports
4. Test connectivity

## Summary

This VLAN configuration provides:
- **6 departmental VLANs** for logical segmentation
- **1 management VLAN** for network device management
- **Trunk links** for VLAN traffic between switches
- **Inter-VLAN routing** via multilayer switch
- **Scalability** for future growth

The configuration demonstrates best practices for VLAN deployment in a layered network architecture, providing efficient communication, security, and scalability.
