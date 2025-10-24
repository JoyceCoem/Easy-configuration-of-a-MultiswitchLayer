# 🖧 Easy Configuration of a Multiswitch Layer

## 📘 Project Overview
**Easy Configuration of a Multiswitch Layer** is a Cisco Packet Tracer project designed to demonstrate a **multilayer network topology** using a combination of **Layer 2 access switches**, a **Layer 3 (multilayer) switch**, and a **router** providing external connectivity.

This topology showcases how to:
- Configure and manage **VLANs** and **inter-VLAN routing** on a multilayer switch  
- Establish **trunk** and **access** links between network devices  
- Implement **static routing** or **dynamic routing protocols** (e.g., OSPF, EIGRP) between a router and a multilayer switch  
- Understand **hierarchical network design** for scalability and performance  

---

## 🗺️ Topology DiagramExample:  
![Network Topology](./topology-image.png)

---

## ⚙️ Setup Instructions

1. **Download and Install Cisco Packet Tracer**  
   Get the latest version from [Cisco Networking Academy](https://www.netacad.com/portal/resources/packet-tracer).

2. **Clone or Download the Repository**
   ```bash
   git clone https://github.com/yourusername/easy-configuration-of-a-multiswitchLayer.git
   cd easy-configuration-of-a-multiswitchLayer

3.**Open the Topology**
 - Launch Cisco Packet Tracer<br/>
 - Open the provided file:<br/>
 - Easy_Multiswitch_Layer.pkt<br/>
 - Run the Simulation
 - Observe device connectivity, VLAN segmentation, and routing.
 - Experiment with ping and tracert between different VLANs and networks.
  
## 🧩 Configuration Steps

1. **VLAN Configuration on the Multilayer Switch**
    ```bash
    Switch(config)# vlan 10
    Switch(config-vlan)# name HR
    Switch(config)# vlan 20
    Switch(config-vlan)# name IT
    Switch(config)# vlan 30
    Switch(config-vlan)# name Sales
    Switch(config)# exit

2. **Assign Access Ports to VLANs**
   ```bash
    Switch(config)# interface range fa0/1 - 12
    Switch(config-if-range)# switchport mode access
    Switch(config-if-range)# switchport access vlan 10
    Switch(config)# interface range fa0/13 - 24
    Switch(config-if-range)# switchport mode access
    Switch(config-if-range)# switchport access vlan 20

3. **Configure Trunk Links**
   ```bash
    Switch(config)# interface gig0/1
    Switch(config-if)# switchport mode trunk
    Switch(config-if)# switchport trunk allowed vlan all

4. **Inter-VLAN Routing on the Multilayer Switch**
     ```bash
    Switch(config)# interface vlan 10
    Switch(config-if)# ip address 192.168.10.1 255.255.255.0
    Switch(config)# interface vlan 20
    Switch(config-if)# ip address 192.168.20.1 255.255.255.0
    Switch(config)# interface vlan 30
    Switch(config-if)# ip address 192.168.30.1 255.255.255.0
    Switch(config)# ip routing

5. **Router Configuration (Static or Dynamic Routing)**
      ```bash
    Static Routing Example:
    Router(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.2
    Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2
    Router(config)# ip route 192.168.30.0 255.255.255.0 10.0.0.2
    Dynamic Routing Example (OSPF):
    Router(config)# router ospf 1
    Router(config-router)# network 10.0.0.0 0.0.0.255 area 0
    Router(config-router)# network 192.168.0.0 0.0.255.255 area 0

---
## 🎓 Learning Outcomes
  By completing this project, learners will gain practical understanding of:
  - The role of a multilayer switch in enterprise networks
  - How to configure VLANs, access ports, and trunks
  - The function and configuration of inter-VLAN routing
  - The difference between static and dynamic routing
  - Designing a hierarchical, scalable network topology
  
---
## 🧰 Tools and Technologies
- Cisco Packet Tracer (Version 8.x or later)
- Cisco 3650 Multilayer Switch
- Cisco 2960 Access Switches
- Cisco ISR4331 Router
- End Devices (PCs, Servers, Cloud)

  ---
Developed as part of a Network Fundamentals / Switching & Routing study project.<br/>
Special thanks to Cisco Networking Academy for simulation tools and learning resources.
