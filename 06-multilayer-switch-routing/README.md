
# Lab 06 — Inter-VLAN Routing with a Multilayer Switch

## Objective

Configure a Cisco multilayer switch to perform **Layer 3 routing between multiple VLANs** using Switch Virtual Interfaces (SVIs).

This lab builds on the previous Router-on-a-Stick lab.

Previously, inter-VLAN communication required:

```text
Hosts
  ↓
Layer 2 Switch
  ↓
802.1Q Trunk
  ↓
Router
  ↓
Routing
```

In this lab, the routing function is moved directly into the multilayer switch:

```text
Hosts
  ↓
Multilayer Switch
  ↓
SVIs
  ↓
IP Routing
```

The lab demonstrates:

* VLAN creation
* Access-port assignment
* Layer 2 switching
* Switch Virtual Interfaces (SVIs)
* Layer 3 IP addressing on a switch
* Enabling `ip routing`
* Connected routes
* Default gateways
* Inter-VLAN routing
* MAC address learning
* Layer 2 and Layer 3 forwarding in the same device
* The difference between a Layer 2 switch and a multilayer switch

---

# 1. Enterprise Scenario

A company has three departments that require logical network separation:

| VLAN | Department | Network           | Default Gateway |
| ---: | ---------- | ----------------- | --------------- |
|   10 | USERS      | `192.168.10.0/24` | `192.168.10.1`  |
|   20 | FINANCE    | `192.168.20.0/24` | `192.168.20.1`  |
|   30 | IT         | `192.168.30.0/24` | `192.168.30.1`  |

Each VLAN represents a separate Layer 2 broadcast domain.

The business requirement is that the departments remain segmented at Layer 2 while still being able to communicate through Layer 3 routing.

Instead of deploying a separate router, the multilayer switch itself provides the Layer 3 gateway for each VLAN.

---

# 2. Topology

![Multilayer switch topology](screenshots/01-toppology.png)

The physical design consists of one Cisco 3560 multilayer switch and endpoint devices distributed across three VLANs.

```text
                         MLS1
                ┌────────────────────┐
                │  Cisco 3560        │
                │  Multilayer Switch │
                │                    │
                │   IP Routing       │
                │       │            │
                │   ┌───┼───┐        │
                │   │   │   │        │
                │ SVI10 SVI20 SVI30  │
                └──┬──┬──┬──┬──┬──┬──┘
                   │  │  │  │  │  │
                  PC0 PC1 PC2 PC3 PC4 PC5
                   │     │     │     │
                 VLAN10 VLAN20 VLAN30
```

The switch performs both:

```text
Layer 2 switching
+
Layer 3 routing
```

---

# 3. VLAN Design

The switch was configured with three departmental VLANs:

```text
VLAN 10 → USERS
VLAN 20 → FINANCE
VLAN 30 → IT
```

The VLANs are deliberately separated into different IP networks:

```text
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
```

This gives the design three independent broadcast domains.

---

# 4. Access Port Assignment

The endpoint ports were configured as access ports.

```text
Fa0/1 → VLAN 10
Fa0/2 → VLAN 10

Fa0/3 → VLAN 20
Fa0/4 → VLAN 20

Fa0/5 → VLAN 30
Fa0/6 → VLAN 30
```

![Access-port configuration](screenshots/03-access-port-configuration.png)

This means the switch associates traffic arriving on each interface with its assigned VLAN.

The endpoints therefore do not need to understand trunking.

They simply send normal Ethernet frames into an access port.

---

# 5. Layer 2 Behavior Before Routing

At the beginning of the lab, MLS1 behaved like a traditional Layer 2 switch.

Within the same VLAN:

```text
PC0 ↔ PC1
```

worked.

Likewise:

```text
PC2 ↔ PC3
```

worked.

However, communication between:

```text
VLAN 10 ↔ VLAN 20
```

failed.

The reason is fundamental:

> A normal Layer 2 switching process forwards Ethernet frames within a VLAN but does not route packets between different IP networks.

This is the same Layer 2 limitation observed in the earlier VLAN and trunking labs.

---

# 6. The New Concept — SVI

The major new concept introduced in this lab is the **Switch Virtual Interface**, or SVI.

An SVI is a logical Layer 3 interface associated with a VLAN.

Instead of placing the gateway on an external router, the multilayer switch itself owns the gateway address.

The design uses:

```text
VLAN 10
   ↓
SVI VLAN 10
   ↓
192.168.10.1

VLAN 20
   ↓
SVI VLAN 20
   ↓
192.168.20.1

VLAN 30
   ↓
SVI VLAN 30
   ↓
192.168.30.1
```

![SVI configuration](screenshots/04-svi-configuration.png)

This means the switch now has Layer 3 interfaces that represent the three VLANs.

---

# 7. SVI vs Router Subinterface

This lab introduces an important comparison with Lab 05.

### Router-on-a-Stick

```text
R1
│
├── G0/0.10 → VLAN 10
└── G0/0.20 → VLAN 20
```

The router performs the routing.

### Multilayer Switch

```text
MLS1
│
├── Vlan10 → VLAN 10
├── Vlan20 → VLAN 20
└── Vlan30 → VLAN 30
```

The switch itself performs the routing.

The physical architecture is therefore simpler:

```text
Router-on-a-Stick:

Switch ─── trunk ─── Router
                  ↓
                routing
```

versus:

```text
Multilayer switch:

Endpoints ─── MLS1
                ↓
              routing
```

---

# 8. Enabling Layer 3 Routing

The multilayer switch was explicitly configured with:

```cisco
ip routing
```

![IP routing enabled](screenshots/05-ip-routing-enabled.png)

This command is the critical difference between using MLS1 only as a Layer 2 switch and allowing it to perform Layer 3 routing.

Without:

```cisco
ip routing
```

the switch could have SVIs with IP addresses but would not function as a router between those VLAN networks.

With:

```cisco
ip routing
```

MLS1 can make Layer 3 forwarding decisions.

---

# 9. Why `ip routing` Matters

Think of the switch in two stages.

### Layer 2 capability

```text
MAC addresses
VLANs
Access ports
STP
Ethernet forwarding
```

### Layer 3 capability

```text
IP addresses
SVIs
Routing table
Inter-VLAN forwarding
```

The command:

```cisco
ip routing
```

activates the Layer 3 forwarding function.

The resulting architecture is:

```text
                 MLS1
        ┌────────────────────┐
        │                    │
VLAN 10 ┤ SVI 192.168.10.1  │
        │        ↕           │
VLAN 20 ┤ SVI 192.168.20.1  │
        │        ↕           │
VLAN 30 ┤ SVI 192.168.30.1  │
        │                    │
        │    IP ROUTING      │
        └────────────────────┘
```

---

# 10. Interface Verification

The switch's Layer 3 interfaces were verified using:

```cisco
show ip interface brief
```

![Interface status](screenshots/06-interface-status.png)

The intended state is:

```text
Vlan10    192.168.10.1    up    up
Vlan20    192.168.20.1    up    up
Vlan30    192.168.30.1    up    up
```

The significance of `up/up` is that the SVI is operational and available as a Layer 3 gateway.

The physical access interfaces also remain part of the Layer 2 switching environment.

---

# 11. Routing Table Verification

The multilayer switch was then checked with:

```cisco
show ip route
```

![Routing table](screenshots/07-routing-table.png)

The routing table should contain directly connected networks similar to:

```text
C 192.168.10.0/24 → Vlan10
C 192.168.20.0/24 → Vlan20
C 192.168.30.0/24 → Vlan30
```

`C` means:

> Connected

This means MLS1 automatically knows that these networks are directly attached to its SVIs.

No static routes are required for communication between these directly connected VLAN networks.

---

# 12. Default Gateway Behavior

Each VLAN uses the corresponding SVI as its default gateway.

Example:

```text
PC0
192.168.10.10/24
Gateway: 192.168.10.1
```

and:

```text
PC2
192.168.20.10/24
Gateway: 192.168.20.1
```

When an endpoint needs to communicate with a host outside its local subnet, it forwards the packet to the default gateway.

The multilayer switch receives the packet and makes a Layer 3 forwarding decision.

---

# 13. Gateway Verification

PC0 was tested against the VLAN 10 gateway:

```text
ping 192.168.10.1
```

![PC0 gateway ping](screenshots/08-pc0-gateway-ping.png)

The successful result proves that PC0 can reach the SVI associated with VLAN 10.

The path is:

```text
PC0
  ↓
Fa0/1
  ↓
VLAN 10
  ↓
SVI VLAN 10
192.168.10.1
```

This confirms that the Layer 2 and Layer 3 path to the local gateway is operational.

---

# 14. Inter-VLAN Routing

The most important test was communication between hosts in different VLANs.

For example:

```text
PC0
192.168.10.10
VLAN 10
```

communicated successfully with a host in VLAN 20.

![Inter-VLAN routing](screenshots/09-inter-vlan-routing.png)

The traffic path is conceptually:

```text
PC0
192.168.10.10
     │
     ▼
VLAN 10
     │
     ▼
SVI 10
192.168.10.1
     │
     │ Layer 3 routing
     ▼
SVI 20
192.168.20.1
     │
     ▼
VLAN 20
     │
     ▼
PC2
192.168.20.10
```

This proves that MLS1 is performing routing between VLANs.

---

# 15. Why This Is Different From the Earlier Labs

### Lab 03

We created VLANs.

```text
VLAN 10
VLAN 20
```

Result:

```text
Segmentation
```

### Lab 04

We introduced trunking.

```text
VLAN 10
VLAN 20
   ↓
802.1Q trunk
```

Result:

```text
VLAN transport
```

### Lab 05

We introduced Router-on-a-Stick.

```text
VLANs
  ↓
Switch
  ↓
Trunk
  ↓
Router
```

Result:

```text
Inter-VLAN routing
```

### Lab 06

We move the routing function into the switch itself.

```text
VLANs
  ↓
Multilayer Switch
  ↓
SVIs
  ↓
ip routing
```

Result:

```text
Inter-VLAN routing without an external router
```

This progression demonstrates how the role of each component changed across the labs.

---

# 16. MAC Address Learning Still Exists

Layer 3 switching does not remove Layer 2 switching.

MLS1 continues to learn Ethernet MAC addresses.

![MAC address table](screenshots/10-mac-address-table.png)

This demonstrates that the multilayer switch is performing both functions simultaneously:

```text
Layer 2:
MAC learning + VLAN switching

Layer 3:
Routing between IP networks
```

This is why the device is called a **multilayer switch**.

---

# 17. Layer 2 + Layer 3 in the Same Device

The completed architecture can be viewed as:

```text
                     MLS1
          ┌────────────────────────┐
          │                        │
          │     Layer 3 Routing    │
          │          ↕             │
          │   ┌──────┼──────┐      │
          │   │      │      │      │
          │ SVI10  SVI20  SVI30    │
          │   │      │      │      │
          │ VLAN10 VLAN20 VLAN30   │
          │   │      │      │      │
          │ Layer 2 Switching      │
          └────────────────────────┘
```

The same physical device therefore participates in both layers.

---

# 18. Verification Commands

The following commands were used:

### VLAN verification

```cisco
show vlan brief
```

### Access-port verification

```cisco
show running-config
```

### SVI verification

```cisco
show running-config
show ip interface brief
```

### Routing verification

```cisco
show ip route
```

### MAC learning

```cisco
show mac address-table dynamic
```

### General configuration verification

```cisco
show running-config
```

---

# 19. Verification Matrix

| Requirement                     | Result |
| ------------------------------- | ------ |
| VLAN 10 configured              | PASS   |
| VLAN 20 configured              | PASS   |
| VLAN 30 configured              | PASS   |
| Access ports correctly assigned | PASS   |
| SVI VLAN 10 configured          | PASS   |
| SVI VLAN 20 configured          | PASS   |
| SVI VLAN 30 configured          | PASS   |
| `ip routing` enabled            | PASS   |
| SVIs operational                | PASS   |
| Connected routes present        | PASS   |
| PC0 reaches gateway             | PASS   |
| Same-VLAN communication         | PASS   |
| Inter-VLAN communication        | PASS   |
| MAC learning                    | PASS   |

---

# 20. Interview-Level Questions

### What is an SVI?

An SVI is a logical Layer 3 interface associated with a VLAN. It can provide the default gateway for hosts in that VLAN.

### Why is `ip routing` required?

Because the multilayer switch must be enabled to perform Layer 3 routing between the directly connected VLAN networks.

### How is this different from Router-on-a-Stick?

Router-on-a-Stick uses an external router with subinterfaces. A multilayer switch uses SVIs and performs the routing internally.

### Can a multilayer switch still perform Layer 2 switching?

Yes. It continues to support VLANs, access ports, MAC learning, and other Layer 2 functions while also performing Layer 3 routing.

### Why doesn't each VLAN need a physical router interface?

Because each VLAN has a logical SVI that provides its Layer 3 gateway.

### What does the routing table prove?

It proves that the multilayer switch has Layer 3 knowledge of the VLAN networks and can make forwarding decisions for them.

---

# 21. Final Result

The final design is:

```text
                    MLS1
          ┌──────────────────────┐
          │   MULTILAYER SWITCH  │
          │                      │
          │      IP ROUTING      │
          │          ↕           │
          │  SVI10  SVI20  SVI30 │
          │   .10.1   .20.1 .30.1│
          └───┬──────┬──────┬────┘
              │      │      │
            VLAN10 VLAN20 VLAN30
              │      │      │
             USERS FINANCE  IT
```

The lab successfully demonstrates:

```text
VLANs
  ↓
Layer 2 segmentation
  ↓
SVIs
  ↓
Default gateways
  ↓
ip routing
  ↓
Routing table
  ↓
Inter-VLAN communication
```

## Conclusion

This lab demonstrates the transition from a traditional Layer 2 access switch to a multilayer switching architecture.

The multilayer switch maintains VLAN-based Layer 2 segmentation while simultaneously providing Layer 3 routing between those VLANs.

The successful gateway and inter-VLAN connectivity tests, together with the SVI status and routing-table evidence, verify that the design is working end-to-end.

The key concept is:

> **A multilayer switch can perform both Layer 2 switching and Layer 3 routing, using SVIs as the gateways for VLANs.**
