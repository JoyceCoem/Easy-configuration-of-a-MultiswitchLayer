# Network Diagram

## Detailed Network Topology Diagram

```
                            ┌──────────────┐
                            │    Cloud     │
                            │  (Internet)  │
                            └──────┬───────┘
                                   │
                            GigabitEthernet0/1
                                   │
                    ┌──────────────┴────────────────┐
                    │         Router (2911)         │
                    │    GigabitEthernet0/0         │
                    │    IP: 10.0.0.1/30            │
                    │    NAT/PAT Enabled            │
                    └──────────────┬────────────────┘
                                   │
                            GigabitEthernet0/1
                            IP: 10.0.0.2/30
                                   │
           ╔═══════════════════════╩════════════════════════╗
           ║         Multilayer Switch (3560)               ║
           ║         Core Layer - Layer 3 Routing           ║
           ║                                                ║
           ║  SVIs (Switch Virtual Interfaces):            ║
           ║  - VLAN 10: 192.168.10.1/24                   ║
           ║  - VLAN 20: 192.168.20.1/24                   ║
           ║  - VLAN 30: 192.168.30.1/24                   ║
           ║  - VLAN 40: 192.168.40.1/24                   ║
           ║  - VLAN 50: 192.168.50.1/24                   ║
           ║  - VLAN 60: 192.168.60.1/24                   ║
           ║  - VLAN 99: 192.168.99.1/24 (Management)      ║
           ╚═══════╦════════════╦════════════╦══════════════╝
                   │            │            │
            Fa0/1  │     Fa0/2  │     Fa0/3  │
           (Trunk) │    (Trunk) │    (Trunk) │
     VLANs 10,20,99│VLANs 30,40,99│VLANs 50,60,99
                   │            │            │
       ┌───────────┴───┐   ┌────┴────────┐  ┌┴────────────┐
       │   Switch1     │   │  Switch2    │  │  Switch3    │
       │   (2960)      │   │  (2960)     │  │  (2960)     │
       │   Access      │   │  Access     │  │  Access     │
       │   Layer       │   │  Layer      │  │  Layer      │
       │               │   │             │  │             │
       │ Mgmt: .99.11  │   │ Mgmt: .99.12│  │ Mgmt: .99.13│
       └───┬───────┬───┘   └───┬─────┬───┘  └───┬─────┬───┘
           │       │           │     │          │     │
        Fa0/1   Fa0/2       Fa0/1 Fa0/2      Fa0/1 Fa0/2
       (Access)(Access)   (Access)(Access) (Access)(Access)
       VLAN 10  VLAN 20    VLAN 30 VLAN 40  VLAN 50 VLAN 60
           │       │           │     │          │     │
           │       │           │     │          │     │
      ┌────┴───┐ ┌┴────┐  ┌───┴──┐ ┌┴───┐  ┌──┴──┐ ┌┴───┐
      │  PC1   │ │ PC2 │  │ PC3  │ │PC4 │  │ PC5 │ │PC6 │
      │  Sales │ │Mktg │  │ Eng  │ │ HR │  │Fin  │ │ IT │
      │        │ │     │  │      │ │    │  │     │ │    │
      │.10.10  │ │.20  │  │.30   │ │.40 │  │.50  │ │.60 │
      │  /24   │ │.10  │  │.10   │ │.10 │  │.10  │ │.10 │
      └────────┘ │/24  │  │/24   │ │/24 │  │/24  │ │/24 │
                 └─────┘  └──────┘ └────┘  └─────┘ └────┘
```

## Layer Structure

```
┌────────────────────────────────────────────────────────┐
│                     LAYER 1: BORDER                    │
│  ┌──────────┐              ┌────────┐                  │
│  │  Cloud   │──────────────│ Router │                  │
│  └──────────┘              └────────┘                  │
│  External Connectivity & Internet Gateway              │
└────────────────────────────────────────────────────────┘
                              │
                              │
┌────────────────────────────┼────────────────────────────┐
│                     LAYER 2: CORE                       │
│                  ┌────────────────────┐                 │
│                  │ Multilayer Switch  │                 │
│                  │  Inter-VLAN Routing│                 │
│                  └────────────────────┘                 │
│  Routing, VLAN Gateways, Central Point                 │
└────────────────────────────┬────────────────────────────┘
                   ┌─────────┴──────────┐
                   │                    │
┌──────────────────┼────────────────────┼─────────────────┐
│             LAYER 3: ACCESS            │                 │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐       │
│   │  Switch1   │  │  Switch2   │  │  Switch3   │       │
│   └────────────┘  └────────────┘  └────────────┘       │
│  End Device Connectivity, VLAN Assignment               │
└────────────────────────────────────────────────────────┘
       │      │         │      │         │      │
       ↓      ↓         ↓      ↓         ↓      ↓
    ┌────┐ ┌────┐   ┌────┐ ┌────┐   ┌────┐ ┌────┐
    │PC1 │ │PC2 │   │PC3 │ │PC4 │   │PC5 │ │PC6 │
    └────┘ └────┘   └────┘ └────┘   └────┘ └────┘
```

## VLAN Distribution

```
┌────────────────────────────────────────────────────────────┐
│              Multilayer Switch (All VLANs)                 │
│  VLAN 10 | VLAN 20 | VLAN 30 | VLAN 40 | VLAN 50 | VLAN 60│
└───┬──────────┬─────────┬──────────┬─────────┬──────────┬───┘
    │          │         │          │         │          │
    ▼          ▼         ▼          ▼         ▼          ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   Switch1     │   │   Switch2     │   │   Switch3     │
├───────────────┤   ├───────────────┤   ├───────────────┤
│ VLAN 10 (PC1) │   │ VLAN 30 (PC3) │   │ VLAN 50 (PC5) │
│ VLAN 20 (PC2) │   │ VLAN 40 (PC4) │   │ VLAN 60 (PC6) │
│ VLAN 99 (Mgmt)│   │ VLAN 99 (Mgmt)│   │ VLAN 99 (Mgmt)│
└───────────────┘   └───────────────┘   └───────────────┘
```

## Trunk Links Detail

```
Multilayer Switch Fa0/1 <==TRUNK==> Switch1 Fa0/24
    │
    ├─ VLAN 10 (Sales)
    ├─ VLAN 20 (Marketing)
    └─ VLAN 99 (Management) ← Native VLAN

Multilayer Switch Fa0/2 <==TRUNK==> Switch2 Fa0/24
    │
    ├─ VLAN 30 (Engineering)
    ├─ VLAN 40 (HR)
    └─ VLAN 99 (Management) ← Native VLAN

Multilayer Switch Fa0/3 <==TRUNK==> Switch3 Fa0/24
    │
    ├─ VLAN 50 (Finance)
    ├─ VLAN 60 (IT)
    └─ VLAN 99 (Management) ← Native VLAN
```

## Traffic Flow Examples

### Example 1: PC1 (VLAN 10) pings PC2 (VLAN 20) - Inter-VLAN

```
Step 1: PC1 sends packet to gateway (192.168.10.1)
        ┌────┐
        │PC1 │ 192.168.10.10 → 192.168.20.10
        └─┬──┘ Dest: PC2, Src: PC1
          │    Gateway: 192.168.10.1
          ↓
Step 2: Switch1 receives on access port (VLAN 10)
        ┌─────────┐
        │Switch1  │ VLAN 10 frame
        └────┬────┘
             │
Step 3: Trunk to Multilayer Switch (802.1Q tagged with VLAN 10)
             ↓
        ╔════════════════╗
        ║ Multilayer SW  ║ Receives on VLAN 10 SVI
        ║ Routes packet  ║ Looks up routing table
        ║ to VLAN 20     ║ Forwards to VLAN 20 SVI
        ╚════┬═══════════╝
             │
Step 4: Trunk to Switch1 (802.1Q tagged with VLAN 20)
             ↓
        ┌─────────┐
        │Switch1  │ VLAN 20 frame
        └────┬────┘
             │
Step 5: Access port delivers to PC2
             ↓
        ┌────┐
        │PC2 │ Receives packet
        └────┘
```

### Example 2: PC1 (VLAN 10) accesses Internet - External

```
Step 1: PC1 sends packet to gateway
        ┌────┐
        │PC1 │ 192.168.10.10 → 8.8.8.8
        └─┬──┘ Gateway: 192.168.10.1
          ↓
Step 2: Switch1 forwards to Multilayer Switch
        ┌─────────┐      ╔════════════════╗
        │Switch1  │─────→║ Multilayer SW  ║
        └─────────┘TRUNK ║ Routes to      ║
                         ║ Router         ║
                         ╚════┬═══════════╝
                              ↓
Step 3: Multilayer Switch routes to Router
                         ┌────────┐
                         │ Router │ 10.0.0.1
                         │ NAT    │ Translates private
                         └────┬───┘ to public IP
                              ↓
Step 4: Router forwards to Internet
                         ┌────────┐
                         │ Cloud  │
                         │Internet│
                         └────────┘
```

### Example 3: PC1 pings PC3 (Different Switches, Different VLANs)

```
PC1 (VLAN 10) → Switch1 → Multilayer Switch (Routes) → Switch2 → PC3 (VLAN 30)

Detailed Flow:
┌────┐   VLAN 10    ┌─────────┐   VLAN 10   ╔════════════════╗
│PC1 │──────────────│Switch1  │──(Trunk)────║ Multilayer SW  ║
└────┘   Access     └─────────┘  802.1Q     ║ Layer 3 Route  ║
                                             ╚════════════════╝
                                                     │ VLAN 30
                                                     │ (Trunk)
                                             ┌───────┴─────┐
                                             │  Switch2    │
                                             └───────┬─────┘
                                                     │ VLAN 30
                                                     │ Access
                                                 ┌───┴──┐
                                                 │ PC3  │
                                                 └──────┘
```

## IP Address Map

```
                    Cloud/Internet
                    (Variable IP)
                          │
                    ┌─────────┐
                    │ Router  │
                    │ G0/1    │ DHCP or 203.0.113.1/24
                    │ G0/0    │ 10.0.0.1/30
                    └─────────┘
                          │
                ┌─────────┴──────────┐
                │ Multilayer Switch  │
                │ G0/1: 10.0.0.2/30  │
                │                    │
                │ SVIs:              │
                │ V10: .10.1  V20:.20.1
                │ V30: .30.1  V40:.40.1
                │ V50: .50.1  V60:.60.1
                │ V99: .99.1 (Mgmt)  │
                └────────┬───────────┘
         ┌───────────────┼──────────────┐
         │               │              │
    ┌────┴────┐    ┌─────┴────┐   ┌────┴────┐
    │Switch1  │    │Switch2   │   │Switch3  │
    │.99.11   │    │.99.12    │   │.99.13   │
    └─┬─────┬─┘    └─┬──────┬─┘   └─┬─────┬─┘
      │     │        │      │       │     │
   ┌──┴──┐┌─┴──┐ ┌──┴──┐┌──┴──┐┌──┴──┐┌─┴──┐
   │PC1  ││PC2 │ │PC3  ││PC4  ││PC5  ││PC6 │
   │.10  ││.20 │ │.30  ││.40  ││.50  ││.60 │
   │.10  ││.10 │ │.10  ││.10  ││.10  ││.10 │
   └─────┘└────┘ └─────┘└─────┘└─────┘└────┘
```

## Port Assignments

### Multilayer Switch (3560)
```
┌────────────────────────────────┐
│    Multilayer Switch Ports     │
├────────────────────────────────┤
│ Fa0/1  → Switch1 (Trunk)       │
│ Fa0/2  → Switch2 (Trunk)       │
│ Fa0/3  → Switch3 (Trunk)       │
│ Gi0/1  → Router (Layer 3)      │
└────────────────────────────────┘
```

### Switch1 (2960)
```
┌────────────────────────────────┐
│      Switch1 Ports             │
├────────────────────────────────┤
│ Fa0/1  → PC1 (VLAN 10)         │
│ Fa0/2  → PC2 (VLAN 20)         │
│ Fa0/24 → Multilayer SW (Trunk) │
└────────────────────────────────┘
```

### Switch2 (2960)
```
┌────────────────────────────────┐
│      Switch2 Ports             │
├────────────────────────────────┤
│ Fa0/1  → PC3 (VLAN 30)         │
│ Fa0/2  → PC4 (VLAN 40)         │
│ Fa0/24 → Multilayer SW (Trunk) │
└────────────────────────────────┘
```

### Switch3 (2960)
```
┌────────────────────────────────┐
│      Switch3 Ports             │
├────────────────────────────────┤
│ Fa0/1  → PC5 (VLAN 50)         │
│ Fa0/2  → PC6 (VLAN 60)         │
│ Fa0/24 → Multilayer SW (Trunk) │
└────────────────────────────────┘
```

### Router (2911)
```
┌────────────────────────────────┐
│       Router Ports             │
├────────────────────────────────┤
│ Gi0/0  → Multilayer SW         │
│          10.0.0.1/30           │
│ Gi0/1  → Cloud/Internet        │
│          DHCP or Static        │
└────────────────────────────────┘
```

## Network Segments

```
┌─────────────────────────────────────────────────────────┐
│ Segment 1: Internal VLANs (192.168.0.0/16)            │
│  - 192.168.10.0/24 (VLAN 10 - Sales)                   │
│  - 192.168.20.0/24 (VLAN 20 - Marketing)               │
│  - 192.168.30.0/24 (VLAN 30 - Engineering)             │
│  - 192.168.40.0/24 (VLAN 40 - HR)                      │
│  - 192.168.50.0/24 (VLAN 50 - Finance)                 │
│  - 192.168.60.0/24 (VLAN 60 - IT)                      │
│  - 192.168.99.0/24 (VLAN 99 - Management)              │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ Segment 2: Router Link (10.0.0.0/30)                   │
│  - 10.0.0.1 (Router)                                    │
│  - 10.0.0.2 (Multilayer Switch)                         │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ Segment 3: External (Cloud/Internet)                    │
│  - Variable addressing (DHCP or Static)                 │
│  - NAT/PAT at Router                                    │
└─────────────────────────────────────────────────────────┘
```

## Legend

```
═══  Multilayer Switch (Layer 3 Device)
───  Layer 2 Switch or Link
<==  Trunk Link (Multiple VLANs)
→    Access Link (Single VLAN)
│    Physical Connection
↓    Traffic Flow Direction
```

---

**Note:** This diagram represents the logical and physical topology of the multiswitch layer network. All connections use appropriate Cisco IOS configurations for optimal performance and security.
