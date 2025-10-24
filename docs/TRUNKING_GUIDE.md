# Trunk Configuration and Inter-VLAN Routing Guide

## Table of Contents
1. [Understanding Trunk Links](#understanding-trunk-links)
2. [802.1Q Encapsulation](#8021q-encapsulation)
3. [Trunk Configuration](#trunk-configuration)
4. [Inter-VLAN Routing](#inter-vlan-routing)
5. [Native VLAN](#native-vlan)
6. [Best Practices](#best-practices)
7. [Troubleshooting](#troubleshooting)

## Understanding Trunk Links

### What is a Trunk?

A **trunk link** is a point-to-point link between two network devices that carries traffic for multiple VLANs. Instead of requiring a separate physical link for each VLAN, a trunk uses VLAN tagging to identify which VLAN each frame belongs to.

### Why Use Trunks?

1. **Efficiency**: One physical link carries multiple VLANs
2. **Scalability**: Easy to add new VLANs without new cables
3. **Cost-Effective**: Reduces cabling requirements
4. **Flexibility**: Dynamic VLAN management

### Trunk vs Access Ports

| Feature | Access Port | Trunk Port |
|---------|------------|------------|
| VLANs | Single VLAN | Multiple VLANs |
| Connection | End devices (PCs, printers) | Switches, routers |
| Tagging | No VLAN tags | VLAN tags (802.1Q) |
| Configuration | Simple | More complex |

## 802.1Q Encapsulation

### VLAN Tagging

802.1Q is the IEEE standard for VLAN tagging on Ethernet networks.

**Frame Structure:**
```
[Dest MAC | Source MAC | 802.1Q Tag | Type | Data | FCS]
                         └─ 4 bytes ─┘
```

**802.1Q Tag (4 bytes):**
- **TPID (Tag Protocol ID)**: 0x8100 (2 bytes)
- **PCP (Priority Code Point)**: QoS priority (3 bits)
- **DEI (Drop Eligible Indicator)**: For congestion management (1 bit)
- **VLAN ID**: VLAN identifier (12 bits) - Supports 4096 VLANs

### Native VLAN

The **native VLAN** is the VLAN for untagged traffic on a trunk link:
- Frames on native VLAN are NOT tagged
- Must match on both ends of trunk
- Default is VLAN 1 (change for security)

## Trunk Configuration

### Our Network Trunk Links

1. **Multilayer Switch ↔ Switch1**: Carries VLANs 10, 20, 99
2. **Multilayer Switch ↔ Switch2**: Carries VLANs 30, 40, 99
3. **Multilayer Switch ↔ Switch3**: Carries VLANs 50, 60, 99
4. **Multilayer Switch ↔ Router**: Carries all VLANs

### Configuring Trunk on Multilayer Switch

```cisco
enable
configure terminal

! Trunk to Switch1
interface FastEthernet0/1
 description Trunk to Switch1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit

! Trunk to Switch2
interface FastEthernet0/2
 description Trunk to Switch2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 30,40,99
 no shutdown
 exit

! Trunk to Switch3
interface FastEthernet0/3
 description Trunk to Switch3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 50,60,99
 no shutdown
 exit
```

### Configuring Trunk on Access Switch

```cisco
enable
configure terminal

! Trunk to Multilayer Switch (Example: Switch1)
interface FastEthernet0/24
 description Trunk to Multilayer Switch
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit
```

**Note:** Access switches (2960) don't require `switchport trunk encapsulation dot1q` as they only support 802.1Q.

### Trunk Configuration Commands Explained

| Command | Purpose |
|---------|---------|
| `switchport trunk encapsulation dot1q` | Sets 802.1Q as encapsulation method |
| `switchport mode trunk` | Configures port as trunk |
| `switchport trunk native vlan 99` | Sets native VLAN to 99 |
| `switchport trunk allowed vlan 10,20,99` | Specifies allowed VLANs |
| `description Trunk to Switch1` | Documents connection |

## Inter-VLAN Routing

### Why Inter-VLAN Routing?

By default, devices in different VLANs cannot communicate because:
- VLANs are separate broadcast domains
- Layer 2 switching doesn't route between VLANs

**Solution:** Use a Layer 3 device (multilayer switch or router) to route between VLANs.

### Methods of Inter-VLAN Routing

#### 1. Router on a Stick (Traditional)
- External router with trunk connection
- Subinterfaces for each VLAN
- Good for small networks
- Router can be bottleneck

#### 2. Multilayer Switch (Modern) ✓ **Our Implementation**
- Switch Virtual Interfaces (SVIs)
- Fast switching at wire speed
- Ideal for enterprise networks
- Scalable and efficient

### Configuring Inter-VLAN Routing

#### Step 1: Enable IP Routing
```cisco
! On Multilayer Switch
enable
configure terminal
ip routing
```

#### Step 2: Create VLANs
```cisco
vlan 10
 name Sales
 exit
vlan 20
 name Marketing
 exit
! Continue for all VLANs...
```

#### Step 3: Configure SVIs
```cisco
! SVI for VLAN 10
interface vlan 10
 description Gateway for Sales VLAN
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

! SVI for VLAN 20
interface vlan 20
 description Gateway for Marketing VLAN
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit

! Repeat for all VLANs...
```

#### Step 4: Configure Trunk Ports
```cisco
interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit
```

### Inter-VLAN Routing Flow

**Example: PC1 (VLAN 10) pings PC2 (VLAN 20)**

1. **PC1** sends packet to default gateway (192.168.10.1)
2. Packet arrives at **Switch1** on access port (VLAN 10)
3. **Switch1** forwards to **Multilayer Switch** via trunk (tagged with VLAN 10)
4. **Multilayer Switch** receives on SVI for VLAN 10
5. **Multilayer Switch** routes to SVI for VLAN 20
6. **Multilayer Switch** forwards via trunk to **Switch1** (tagged with VLAN 20)
7. **Switch1** forwards to **PC2** on access port
8. **PC2** receives packet

```
PC1 (VLAN 10) → Switch1 → [Trunk] → Multilayer Switch (Routing) 
→ [Trunk] → Switch1 → PC2 (VLAN 20)
```

## Native VLAN

### Understanding Native VLAN

- **Purpose**: Handles untagged frames on trunk
- **Default**: VLAN 1
- **Best Practice**: Change to non-default (we use VLAN 99)
- **Requirement**: Must match on both ends of trunk

### Why Change Native VLAN?

1. **Security**: VLAN 1 is well-known, making it a target
2. **Management**: Separate management traffic
3. **Best Practice**: Industry standard recommendation

### Configuring Native VLAN

```cisco
interface FastEthernet0/1
 switchport trunk native vlan 99
 exit
```

### Native VLAN Mismatch

**Symptom:** CDP warnings, inconsistent behavior

**Check:**
```cisco
show interfaces trunk
show interfaces FastEthernet0/1 switchport
```

**Fix:** Ensure both sides have same native VLAN

## Best Practices

### 1. Trunk Configuration

✅ **Do:**
- Use descriptive interface descriptions
- Explicitly configure native VLAN (not VLAN 1)
- Limit allowed VLANs to only necessary ones
- Use `switchport mode trunk` (disable DTP negotiation)
- Document trunk connections

❌ **Don't:**
- Use VLAN 1 as native VLAN
- Allow all VLANs when not needed
- Rely on automatic trunk negotiation
- Leave trunks without descriptions

### 2. Inter-VLAN Routing

✅ **Do:**
- Enable IP routing: `ip routing`
- Configure descriptive SVI descriptions
- Use consistent IP addressing scheme
- Verify all SVIs are up
- Document gateway addresses

❌ **Don't:**
- Forget to enable `no shutdown` on SVIs
- Use overlapping IP ranges
- Skip IP routing configuration
- Use inconsistent subnet masks

### 3. VLAN Management

✅ **Do:**
- Create VLANs before assigning ports
- Use meaningful VLAN names
- Maintain VLAN documentation
- Use separate management VLAN

❌ **Don't:**
- Use only VLAN numbers without names
- Create VLANs ad-hoc without planning
- Use VLAN 1 for production traffic

### 4. Security

✅ **Do:**
- Change native VLAN from default
- Prune unnecessary VLANs from trunks
- Disable unused ports
- Implement port security on access ports

❌ **Don't:**
- Leave native VLAN as VLAN 1
- Allow all VLANs on all trunks
- Leave unused ports enabled

## Troubleshooting

### Issue 1: Trunk Not Forming

**Symptoms:**
- `show interfaces trunk` shows no trunk
- Devices on different switches can't communicate

**Diagnosis:**
```cisco
show interfaces FastEthernet0/1 switchport
show interfaces FastEthernet0/1 status
```

**Solutions:**
1. **Check Both Sides**: Ensure both configured as trunk
2. **Verify Encapsulation**: Both must use dot1q
3. **Check Physical Link**: Ensure cable connected and port up
4. **Manual Configuration**:
   ```cisco
   interface FastEthernet0/1
   switchport mode trunk
   switchport trunk encapsulation dot1q
   no shutdown
   ```

### Issue 2: Native VLAN Mismatch

**Symptoms:**
- CDP/LLDP warnings
- Inconsistent connectivity
- Some VLANs work, others don't

**Diagnosis:**
```cisco
show interfaces trunk
show cdp neighbors detail
```

**Solution:**
```cisco
! On both sides of trunk
interface FastEthernet0/1
 switchport trunk native vlan 99
 exit
```

### Issue 3: VLAN Not Allowed on Trunk

**Symptoms:**
- Devices in specific VLAN can't communicate across switches

**Diagnosis:**
```cisco
show interfaces trunk
```

Look for "VLANs allowed on trunk" line

**Solution:**
```cisco
interface FastEthernet0/1
 switchport trunk allowed vlan add <vlan-id>
 ! Or set entire list:
 switchport trunk allowed vlan 10,20,30,99
 exit
```

### Issue 4: Inter-VLAN Routing Not Working

**Symptoms:**
- Devices in same VLAN can communicate
- Devices in different VLANs cannot communicate
- Can ping gateway but not other VLANs

**Diagnosis:**
```cisco
show ip route
show ip interface brief
show vlan brief
```

**Solutions:**
1. **Enable IP Routing**:
   ```cisco
   configure terminal
   ip routing
   ```

2. **Check SVI Status**:
   ```cisco
   show ip interface brief
   ! Look for SVIs - should be up/up
   ```

3. **Verify VLAN Exists**:
   ```cisco
   show vlan brief
   ! VLAN must exist before SVI works
   ```

4. **Enable SVI**:
   ```cisco
   interface vlan 10
   no shutdown
   exit
   ```

### Issue 5: Trunk Carrying Wrong VLANs

**Symptoms:**
- Unexpected VLAN traffic on trunk
- Security concerns

**Diagnosis:**
```cisco
show interfaces trunk
show vlan brief
```

**Solution:**
```cisco
interface FastEthernet0/1
 ! Remove all allowed VLANs and add specific ones
 switchport trunk allowed vlan none
 switchport trunk allowed vlan 10,20,99
 exit
```

## Verification Commands

### Trunk Status
```cisco
show interfaces trunk
show interfaces <interface-id> switchport
show interfaces <interface-id> status
```

### VLAN Information
```cisco
show vlan brief
show vlan id <vlan-id>
```

### Inter-VLAN Routing
```cisco
show ip route
show ip interface brief
show interfaces vlan <vlan-id>
```

### Detailed Interface Info
```cisco
show interfaces <interface-id>
show running-config interface <interface-id>
```

## Advanced Topics

### DTP (Dynamic Trunking Protocol)

DTP automatically negotiates trunking:
- **desirable**: Actively tries to form trunk
- **auto**: Forms trunk if other side is desirable
- **trunk**: Permanently trunk (no negotiation)

**Best Practice:** Disable DTP for security
```cisco
interface FastEthernet0/1
 switchport mode trunk
 switchport nonegotiate
 exit
```

### VTP (VLAN Trunking Protocol)

VTP synchronizes VLAN database across switches:
- **Server Mode**: Can create/modify/delete VLANs
- **Client Mode**: Receives VLAN info from server
- **Transparent Mode**: Doesn't participate, forwards VTP

**Best Practice:** Use transparent mode or VTP version 3

### Pruning VLANs

Prevents unnecessary VLAN traffic on trunks:
```cisco
! Manually specify allowed VLANs
interface FastEthernet0/1
 switchport trunk allowed vlan 10,20
 exit

! Or use VTP pruning
vtp pruning
```

## Summary

This guide covered:
- **Trunk configuration** with 802.1Q encapsulation
- **Inter-VLAN routing** using multilayer switch
- **Native VLAN** configuration and best practices
- **Troubleshooting** common trunk and routing issues

Key takeaways:
1. Trunks use VLAN tagging (802.1Q) to carry multiple VLANs
2. Inter-VLAN routing requires Layer 3 device (multilayer switch)
3. Native VLAN should be changed from default (VLAN 1)
4. Limit allowed VLANs on trunks to only necessary ones
5. Always enable `ip routing` and `no shutdown` on SVIs

For more information, refer to:
- [VLAN_GUIDE.md](VLAN_GUIDE.md) - VLAN configuration details
- [NETWORK_TOPOLOGY.md](NETWORK_TOPOLOGY.md) - Complete network design
- [SETUP_GUIDE.md](SETUP_GUIDE.md) - Step-by-step setup
