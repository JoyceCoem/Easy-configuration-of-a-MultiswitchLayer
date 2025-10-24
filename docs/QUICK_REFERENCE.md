# Quick Reference Guide

## Network Information

### Device Inventory
- 1x Multilayer Switch (Cisco 3560)
- 3x Access Switches (Cisco 2960)
- 1x Router (Cisco 2911)
- 6x PCs
- 1x Cloud-PT

### VLAN Quick Reference

| VLAN | Name        | Subnet           | Gateway       | Device |
|------|-------------|------------------|---------------|--------|
| 10   | Sales       | 192.168.10.0/24  | 192.168.10.1  | PC1    |
| 20   | Marketing   | 192.168.20.0/24  | 192.168.20.1  | PC2    |
| 30   | Engineering | 192.168.30.0/24  | 192.168.30.1  | PC3    |
| 40   | HR          | 192.168.40.0/24  | 192.168.40.1  | PC4    |
| 50   | Finance     | 192.168.50.0/24  | 192.168.50.1  | PC5    |
| 60   | IT          | 192.168.60.0/24  | 192.168.60.1  | PC6    |
| 99   | Management  | 192.168.99.0/24  | 192.168.99.1  | Mgmt   |

### IP Address Assignments

#### PCs
- **PC1**: 192.168.10.10/24, GW: 192.168.10.1
- **PC2**: 192.168.20.10/24, GW: 192.168.20.1
- **PC3**: 192.168.30.10/24, GW: 192.168.30.1
- **PC4**: 192.168.40.10/24, GW: 192.168.40.1
- **PC5**: 192.168.50.10/24, GW: 192.168.50.1
- **PC6**: 192.168.60.10/24, GW: 192.168.60.1

#### Switches
- **Switch1 (Management)**: 192.168.99.11/24
- **Switch2 (Management)**: 192.168.99.12/24
- **Switch3 (Management)**: 192.168.99.13/24

#### Multilayer Switch & Router
- **Multilayer Switch GigabitEthernet0/1**: 10.0.0.2/30
- **Router GigabitEthernet0/0**: 10.0.0.1/30
- **Router GigabitEthernet0/1**: DHCP or 203.0.113.1/24

### Trunk Links

| Source | Interface | Destination | VLANs Allowed | Native VLAN |
|--------|-----------|-------------|---------------|-------------|
| Multilayer Switch | Fa0/1 | Switch1 Fa0/24 | 10,20,99 | 99 |
| Multilayer Switch | Fa0/2 | Switch2 Fa0/24 | 30,40,99 | 99 |
| Multilayer Switch | Fa0/3 | Switch3 Fa0/24 | 50,60,99 | 99 |

## Essential Commands

### Basic Navigation
```cisco
enable                          # Enter privileged mode
configure terminal              # Enter configuration mode
exit                            # Exit current mode
end                             # Return to privileged mode
```

### Save Configuration
```cisco
write memory                    # Save configuration
copy running-config startup-config  # Alternative save command
```

### Show Commands
```cisco
show vlan brief                 # Display VLAN summary
show interfaces trunk           # Display trunk status
show ip interface brief         # Display interface IP summary
show ip route                   # Display routing table
show running-config             # Display current configuration
show interfaces status          # Display interface status
show port-security              # Display port security info
show mac address-table          # Display MAC address table
```

### VLAN Commands
```cisco
# Create VLAN
vlan 10
 name Sales
 exit

# Assign port to VLAN
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 exit

# Create SVI (on multilayer switch)
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
```

### Trunk Commands
```cisco
# Configure trunk port
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk encapsulation dot1q  # Only on multilayer switch
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit
```

### Inter-VLAN Routing
```cisco
# Enable routing (on multilayer switch)
ip routing

# Verify routing
show ip route
show ip interface brief
```

### Port Security
```cisco
interface FastEthernet0/1
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 exit
```

### Interface Configuration
```cisco
# Basic interface config
interface FastEthernet0/1
 description PC1 - Sales Department
 switchport mode access
 switchport access vlan 10
 no shutdown
 exit

# Shutdown interface
interface FastEthernet0/1
 shutdown
 exit

# Enable interface
interface FastEthernet0/1
 no shutdown
 exit
```

## Troubleshooting Quick Guide

### Can't Ping Gateway
```cisco
# Check IP configuration on PC
# Check VLAN assignment
show vlan brief
show interfaces FastEthernet0/1 switchport

# Check SVI status
show ip interface brief
show interfaces vlan 10
```

### Inter-VLAN Communication Fails
```cisco
# Enable IP routing
configure terminal
ip routing
exit

# Check SVIs
show ip interface brief

# Check routing table
show ip route

# Verify trunk
show interfaces trunk
```

### Trunk Not Working
```cisco
# Check trunk status
show interfaces trunk
show interfaces FastEthernet0/24 switchport

# Configure trunk manually
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk native vlan 99
 no shutdown
 exit
```

### Port Security Violation
```cisco
# Check port security
show port-security interface FastEthernet0/1

# Clear violation
configure terminal
interface FastEthernet0/1
 shutdown
 no shutdown
 exit
```

## Testing Commands

### From PC Command Prompt
```bash
ping 192.168.10.1              # Test gateway
ping 192.168.20.10             # Test inter-VLAN
ping 10.0.0.1                  # Test router
ping 8.8.8.8                   # Test internet
ipconfig                       # Display IP configuration
```

### From Switch/Router CLI
```cisco
ping 192.168.10.10             # Test PC connectivity
traceroute 192.168.20.10       # Trace route to destination
show arp                       # Display ARP table
show cdp neighbors             # Display connected Cisco devices
```

## Default Credentials

All devices use the same credentials:
- **Enable Secret**: cisco123
- **Console Password**: cisco
- **VTY Password**: cisco

## Configuration Files

All configuration files are located in the `configs/` directory:
- `multilayer-switch-config.txt` - Core switch configuration
- `switch1-config.txt` - Access switch 1 (VLANs 10, 20)
- `switch2-config.txt` - Access switch 2 (VLANs 30, 40)
- `switch3-config.txt` - Access switch 3 (VLANs 50, 60)
- `router-config.txt` - Router configuration

## Common Issues and Quick Fixes

| Issue | Quick Fix |
|-------|-----------|
| Port in err-disabled state | `shutdown` then `no shutdown` |
| Trunk not forming | Configure both sides as trunk |
| Native VLAN mismatch | Set same native VLAN on both sides |
| Can't ping across VLANs | Enable `ip routing` on multilayer switch |
| SVI down | `no shutdown` on VLAN interface |
| VLAN not in database | Create VLAN first with `vlan <id>` |

## Best Practices Checklist

- [ ] Use descriptive interface descriptions
- [ ] Change native VLAN from VLAN 1
- [ ] Limit allowed VLANs on trunks
- [ ] Use port security on access ports
- [ ] Shutdown unused ports
- [ ] Document all configuration changes
- [ ] Save configurations regularly
- [ ] Test connectivity after changes
- [ ] Use management VLAN separate from data
- [ ] Implement password security

## Traffic Flow Summary

### Same VLAN Communication
PC1 → Switch1 (Layer 2) → PC1 (same VLAN)

### Inter-VLAN Communication
PC1 (VLAN 10) → Switch1 → Multilayer Switch (Routes) → Switch1 → PC2 (VLAN 20)

### External Communication
PC1 → Switch1 → Multilayer Switch → Router → Cloud/Internet

## Physical Connections

### Multilayer Switch
- Fa0/1 → Switch1 Fa0/24 (Trunk)
- Fa0/2 → Switch2 Fa0/24 (Trunk)
- Fa0/3 → Switch3 Fa0/24 (Trunk)
- Gi0/1 → Router Gi0/0

### Switch1
- Fa0/1 → PC1
- Fa0/2 → PC2
- Fa0/24 → Multilayer Switch Fa0/1 (Trunk)

### Switch2
- Fa0/1 → PC3
- Fa0/2 → PC4
- Fa0/24 → Multilayer Switch Fa0/2 (Trunk)

### Switch3
- Fa0/1 → PC5
- Fa0/2 → PC6
- Fa0/24 → Multilayer Switch Fa0/3 (Trunk)

### Router
- Gi0/0 → Multilayer Switch Gi0/1
- Gi0/1 → Cloud Ethernet6

## Resources

- **Full Documentation**: See `docs/` directory
- **Setup Guide**: `docs/SETUP_GUIDE.md`
- **VLAN Guide**: `docs/VLAN_GUIDE.md`
- **Trunking Guide**: `docs/TRUNKING_GUIDE.md`
- **Network Topology**: `docs/NETWORK_TOPOLOGY.md`

---

**Quick Tip**: Always verify your changes with `show` commands before saving configuration!
