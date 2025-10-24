# Network Topology

## Overview
This document describes the network topology for the multiswitch layer configuration using Cisco Packet Tracer.

## Network Architecture

### Layer Structure

```
                    [Cloud]
                       |
                   [Router]
                       |
              [Multilayer Switch]
                /      |      \
               /       |       \
        [Switch1]  [Switch2]  [Switch3]
         /    \      /    \      /    \
       PC1   PC2   PC3   PC4   PC5   PC6
```

## Network Components

### 1. Multilayer Switch (Core Layer)
- **Device**: Cisco 3560 or similar multilayer switch
- **Function**: 
  - Central routing and switching
  - Inter-VLAN routing
  - Trunk link aggregation
  - Gateway for all VLANs
- **IP Address**: 192.168.1.1/24 (Management)

### 2. Access Switches (Access Layer)
- **Device**: Cisco 2960 or similar access switches
- **Quantity**: 3 switches
- **Function**: 
  - End device connectivity
  - VLAN segmentation
  - Port security

#### Switch1 (Access Switch)
- **VLANs**: VLAN 10, VLAN 20
- **Connected PCs**: PC1, PC2

#### Switch2 (Access Switch)
- **VLANs**: VLAN 30, VLAN 40
- **Connected PCs**: PC3, PC4

#### Switch3 (Access Switch)
- **VLANs**: VLAN 50, VLAN 60
- **Connected PCs**: PC5, PC6

### 3. Router (Border)
- **Device**: Cisco 2911 or similar router
- **Function**: 
  - External connectivity
  - Cloud/Internet gateway
  - NAT/PAT services
- **Interfaces**:
  - GigabitEthernet0/0: Connected to Multilayer Switch
  - GigabitEthernet0/1: Connected to Cloud

### 4. End Devices
- **PCs**: 6 workstations distributed across VLANs
- **Assignment**:
  - PC1: VLAN 10 (Sales)
  - PC2: VLAN 20 (Marketing)
  - PC3: VLAN 30 (Engineering)
  - PC4: VLAN 40 (HR)
  - PC5: VLAN 50 (Finance)
  - PC6: VLAN 60 (IT)

### 5. Cloud Connection
- **Purpose**: Simulates external network/Internet
- **Configuration**: Connected to router for external communication

## VLAN Configuration

### VLAN Table

| VLAN ID | Name        | Subnet           | Gateway       | Description              |
|---------|-------------|------------------|---------------|--------------------------|
| 10      | Sales       | 192.168.10.0/24  | 192.168.10.1  | Sales Department         |
| 20      | Marketing   | 192.168.20.0/24  | 192.168.20.1  | Marketing Department     |
| 30      | Engineering | 192.168.30.0/24  | 192.168.30.1  | Engineering Department   |
| 40      | HR          | 192.168.40.0/24  | 192.168.40.1  | Human Resources          |
| 50      | Finance     | 192.168.50.0/24  | 192.168.50.1  | Finance Department       |
| 60      | IT          | 192.168.60.0/24  | 192.168.60.1  | IT Department            |
| 99      | Management  | 192.168.99.0/24  | 192.168.99.1  | Network Management       |

## Trunk Links

### Trunk Connections
1. **Multilayer Switch to Switch1**: Trunk (Carries VLANs 10, 20)
2. **Multilayer Switch to Switch2**: Trunk (Carries VLANs 30, 40)
3. **Multilayer Switch to Switch3**: Trunk (Carries VLANs 50, 60)
4. **Multilayer Switch to Router**: Trunk (All VLANs + Native VLAN 99)

### Trunk Configuration Details
- **Encapsulation**: 802.1Q
- **Native VLAN**: 99
- **Allowed VLANs**: All or specific per trunk
- **DTP Mode**: Trunk (non-negotiate)

## IP Addressing Scheme

### Multilayer Switch Interfaces (SVIs)
- VLAN 10: 192.168.10.1/24
- VLAN 20: 192.168.20.1/24
- VLAN 30: 192.168.30.1/24
- VLAN 40: 192.168.40.1/24
- VLAN 50: 192.168.50.1/24
- VLAN 60: 192.168.60.1/24
- VLAN 99: 192.168.99.1/24

### Router Interfaces
- GigabitEthernet0/0: 10.0.0.1/30 (to Multilayer Switch)
- GigabitEthernet0/1: DHCP from Cloud (or 203.0.113.1/24)

### End Device IPs
- PC1: 192.168.10.10/24 (Gateway: 192.168.10.1)
- PC2: 192.168.20.10/24 (Gateway: 192.168.20.1)
- PC3: 192.168.30.10/24 (Gateway: 192.168.30.1)
- PC4: 192.168.40.10/24 (Gateway: 192.168.40.1)
- PC5: 192.168.50.10/24 (Gateway: 192.168.50.1)
- PC6: 192.168.60.10/24 (Gateway: 192.168.60.1)

## Scalability Features

1. **Hierarchical Design**: Clear separation of core, distribution, and access layers
2. **VLAN Segmentation**: Easy addition of new VLANs and departments
3. **Trunk Capacity**: Supports adding more VLANs without physical changes
4. **Inter-VLAN Routing**: Centralized routing on multilayer switch
5. **Modular Access**: Easy to add more access switches and end devices

## Network Flow

### Same VLAN Communication
- Traffic stays within the access switch (Layer 2 switching)

### Inter-VLAN Communication
1. PC sends packet to default gateway (SVI on Multilayer Switch)
2. Multilayer Switch routes packet to destination VLAN
3. Packet forwarded through trunk to appropriate access switch
4. Access switch delivers to destination PC

### External Communication
1. PC sends packet to default gateway
2. Multilayer Switch routes to Router
3. Router forwards to Cloud/Internet
4. Return traffic follows reverse path
