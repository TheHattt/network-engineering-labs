
# Practice Lab 06B — Static Routing Between Two LANs

## Objective

Build and verify a routed network containing two separate LANs connected by two routers and a point-to-point transit network.

This practice lab introduces the concept of **static routing**.

The main objective is to understand the difference between:

```text
Directly Connected Networks
        vs
Remote Networks
```

and how a router can be manually instructed to reach a remote network through a specific next-hop router.

This lab builds directly on the Layer 3 concepts learned in Lab 06.

---

# 1. Why This Practice Lab Exists

In Lab 06, the multilayer switch knew about its VLAN networks because those networks were directly connected to its SVIs.

The router automatically learned directly connected networks.

The next question was:

> What happens when a router needs to reach a network that is not directly connected to it?

That is the problem solved by static routing.

The final topology demonstrates:

```text
Local Network
      ↓
Router
      ↓
Transit Network
      ↓
Router
      ↓
Remote Network
```

---

# 2. Network Topology

The practice topology contains:

```text
2 PCs
2 switches
2 routers
```

The logical topology is:

```text
PC0 ── SW1 ── R1 ───────── R2 ── SW2 ── PC1
                    │
              Transit Link
               10.0.0.0/30
```

The physical interface assignments are:

```text
R1 G0/0 → SW1 Fa0/24
R1 G0/1 → R2 G0/1

R2 G0/0 → SW2 Fa0/24
```

---

# 3. IP Addressing Plan

The network uses three separate IP networks.

## LAN A

```text
192.168.10.0/24
```

Used by PC0 and R1.

## Transit Network

```text
10.0.0.0/30
```

Used only between R1 and R2.

## LAN B

```text
192.168.20.0/24
```

Used by R2 and PC1.

---

# 4. Device Addressing

## PC0

```text
IP Address:      192.168.10.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1
```

## R1

```text
G0/0 → 192.168.10.1/24
G0/1 → 10.0.0.1/30
```

## R2

```text
G0/1 → 10.0.0.2/30
G0/0 → 192.168.20.1/24
```

## PC1

```text
IP Address:      192.168.20.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.20.1
```

---

# 5. Understanding the Transit Network

The R1-to-R2 network is:

```text
10.0.0.0/30
```

A `/30` provides four addresses:

```text
Network:    10.0.0.0
Host:       10.0.0.1
Host:       10.0.0.2
Broadcast:  10.0.0.3
```

The routers therefore use:

```text
R1 → 10.0.0.1
R2 → 10.0.0.2
```

The transit network is not an end-user LAN.

Its purpose is to connect the two routers.

Conceptually:

```text
LAN A
192.168.10.0/24
      │
      R1
      │
      │ 10.0.0.0/30
      │
      R2
      │
LAN B
192.168.20.0/24
```

---

# 6. Directly Connected Networks

After configuring the interfaces, R1 automatically knows:

```text
C 192.168.10.0/24
C 10.0.0.0/30
```

because these networks are directly connected to R1.

R2 automatically knows:

```text
C 10.0.0.0/30
C 192.168.20.0/24
```

because these networks are directly connected to R2.

The important observation is:

```text
R1 knows its own LAN.
R2 knows its own LAN.
```

But initially:

```text
R1 does NOT know 192.168.20.0/24.
R2 does NOT know 192.168.10.0/24.
```

Those are remote networks.

---

# 7. Initial Routing Problem

Before static routing was configured, PC0 attempted to reach:

```text
192.168.20.10
```

The destination belongs to:

```text
192.168.20.0/24
```

R1 did not initially have that network in its routing table.

Therefore the packet could not be forwarded successfully.

The path effectively stopped at R1:

```text
PC0
 ↓
R1
 ↓
192.168.20.0/24
      ?
```

This demonstrated that having physical connectivity between routers does not automatically mean they know about all remote networks.

---

# 8. The Role of Static Routing

A static route manually tells a router how to reach a remote network.

The logic is:

```text
Destination Network
        +
Next-Hop Router
        ↓
Static Route
```

For R1:

```text
Destination:
192.168.20.0/24

Next Hop:
10.0.0.2
```

This means:

> To reach the 192.168.20.0/24 network, send traffic toward R2 at 10.0.0.2.

For R2:

```text
Destination:
192.168.10.0/24

Next Hop:
10.0.0.1
```

This means:

> To reach the 192.168.10.0/24 network, send traffic toward R1 at 10.0.0.1.

---

# 9. Static Route on R1

R1 was configured with a static route to LAN B.

Conceptually:

```text
192.168.20.0/24
        ↓
    10.0.0.2
        ↓
       R2
```

The routing table then contains:

```text
S 192.168.20.0/24 [1/0] via 10.0.0.2
```

Where:

```text
S = Static
```

This differs from:

```text
C = Connected
```

because the connected network was learned automatically while the static route was manually configured.

---

# 10. Static Route on R2

R2 was configured with the reverse path:

```text
192.168.10.0/24
        ↓
    10.0.0.1
        ↓
       R1
```

R2's routing table then contains:

```text
S 192.168.10.0/24 [1/0] via 10.0.0.1
```

Now both routers have a path to the remote LAN.

---

# 11. Why Both Routes Are Required

A common misunderstanding is:

> "If R1 knows how to get to PC1, communication should work."

Not necessarily.

IP communication requires a return path.

The forward path is:

```text
PC0
 ↓
R1
 ↓
Transit
 ↓
R2
 ↓
PC1
```

The reply must return:

```text
PC1
 ↓
R2
 ↓
Transit
 ↓
R1
 ↓
PC0
```

Therefore:

```text
R1 needs a route to LAN B.
R2 needs a route to LAN A.
```

This was demonstrated directly during the practice lab.

---

# 12. Routing Table Before Static Routing

R1 initially knew:

```text
C 192.168.10.0/24
C 10.0.0.0/30
```

R2 initially knew:

```text
C 192.168.20.0/24
C 10.0.0.0/30
```

The remote networks were missing.

Conceptually:

```text
R1
├── LAN A       ✅
├── Transit     ✅
└── LAN B       ❌

R2
├── Transit     ✅
├── LAN B       ✅
└── LAN A       ❌
```

---

# 13. Routing Table After Static Routing

After configuring the static routes:

R1 knows:

```text
C 192.168.10.0/24
C 10.0.0.0/30
S 192.168.20.0/24 via 10.0.0.2
```

R2 knows:

```text
C 192.168.20.0/24
C 10.0.0.0/30
S 192.168.10.0/24 via 10.0.0.1
```

The complete path is now available.

---

# 14. End-to-End Packet Flow

When PC0 sends traffic to PC1:

```text
PC0
192.168.10.10
      │
      ▼
Default Gateway
192.168.10.1
      │
      ▼
R1
      │
      │ Static route
      │ 192.168.20.0/24
      │ via 10.0.0.2
      ▼
R2
      │
      │ Directly connected
      │ 192.168.20.0/24
      ▼
PC1
192.168.20.10
```

The reply follows the reverse route:

```text
PC1
      │
      ▼
R2
      │
      │ Static route
      │ 192.168.10.0/24
      │ via 10.0.0.1
      ▼
R1
      │
      ▼
PC0
```

---

# 15. Transit Connectivity Verification

The transit link was tested directly.

R1 successfully reached R2:

```text
ping 10.0.0.2
```

The result was:

```text
Success rate is 80 percent (4/5)
```

The initial packet loss was associated with neighbor/ARP resolution behavior in the simulated environment.

The important result was that subsequent packets succeeded, proving R1 and R2 could communicate across the transit network.

---

# 16. End-to-End Connectivity Before Static Routing

Before static routes were added:

```text
PC0 → 192.168.20.10
```

failed.

The failure demonstrated:

```text
Physical connectivity = working
Transit link = working
Remote route = missing
```

This is a useful troubleshooting distinction.

A router can reach the next router but still be unable to reach a remote LAN.

---

# 17. End-to-End Connectivity After Static Routing

After configuring static routes in both directions:

```text
PC0 → 192.168.20.10
```

succeeded.

The reverse test:

```text
PC1 → 192.168.10.10
```

also succeeded.

### Result

**PASS**

This proves:

```text
LAN A
   ↕
R1
   ↕
Transit network
   ↕
R2
   ↕
LAN B
```

is fully reachable in both directions.

---

# 18. Static Route Meaning

The command conceptually means:

```text
"To reach this remote network,
use this next router."
```

For R1:

```text
192.168.20.0/24
        ↓
10.0.0.2
```

For R2:

```text
192.168.10.0/24
        ↓
10.0.0.1
```

This is the foundation of static routing.

---

# 19. Connected vs Static Routes

The routing table now contains two different route types.

## Connected

```text
C 192.168.10.0/24
```

Meaning:

> This network is directly connected to the router.

## Static

```text
S 192.168.20.0/24 via 10.0.0.2
```

Meaning:

> This network was manually configured as reachable through a specific next hop.

This distinction is essential for understanding routing tables.

---

# 20. Administrative Distance

The static route appeared as:

```text
S 192.168.20.0/24 [1/0] via 10.0.0.2
```

The brackets contain:

```text
[Administrative Distance / Metric]
```

For the static route:

```text
Administrative Distance = 1
```

Static routes therefore have a low default administrative distance compared with many dynamic routing protocols.

This will become important later when learning route selection.

---

# 21. Why Routing Is Directional

Routing tables describe where a router should send traffic.

R1's table describes paths from R1.

R2's table describes paths from R2.

Therefore:

```text
R1's route
```

does not automatically create:

```text
R2's route
```

Each router must have appropriate knowledge of the networks it needs to reach.

This is why both static routes were required.

---

# 22. Troubleshooting Process

The troubleshooting sequence used in this practice lab was:

```text
1. Verify PC addressing
        ↓
2. Verify router interfaces
        ↓
3. Verify directly connected routes
        ↓
4. Verify R1 ↔ R2 transit connectivity
        ↓
5. Test remote LAN
        ↓
6. Identify missing route
        ↓
7. Add static route on R1
        ↓
8. Test again
        ↓
9. Identify return-path requirement
        ↓
10. Add static route on R2
        ↓
11. Test both directions
```

This process demonstrates how to isolate a routing problem instead of changing unrelated configurations.

---

# 23. Key Troubleshooting Observation

A particularly useful observation occurred after the R1 static route was added.

Before:

```text
Destination host unreachable
```

After adding the route:

```text
Request timed out
```

The meaning was important.

R1 had stopped rejecting the destination as unknown.

The packet could now move toward R2.

The remaining failure was caused by the missing return route on R2.

This demonstrated that troubleshooting output can reveal **where the packet has progressed in the network**.

---

# 24. Important Mental Model

The routing process can be summarized as:

```text
LOCAL NETWORK
      ↓
DEFAULT GATEWAY
      ↓
ROUTER
      ↓
ROUTING TABLE
      ↓
NEXT HOP
      ↓
TRANSIT NETWORK
      ↓
REMOTE ROUTER
      ↓
REMOTE NETWORK
```

The return traffic performs the reverse journey.

---

# 25. Comparison With Lab 06

Lab 06 taught:

```text
Directly Connected Networks
        ↓
SVIs
        ↓
Multilayer Switch
        ↓
Inter-VLAN Routing
```

This practice lab extends that concept:

```text
Local Network
        ↓
Router
        ↓
Remote Network
        ↓
Static Route
```

The progression is:

```text
Connected Route
      ↓
Static Route
      ↓
Dynamic Route
```

This is the beginning of understanding how routers build and use larger routing tables.

---

# 26. Verification Commands

## Interface status

```cisco
show ip interface brief
```

Purpose:

Verify interfaces are operational and have the intended addresses.

## Routing table

```cisco
show ip route
```

Purpose:

Verify connected and static routes.

## Connectivity

```cisco
ping <destination>
```

Purpose:

Verify reachability between devices.

## Configuration

```cisco
show running-config
```

Purpose:

Inspect configured interface addresses and static routes.

---

# 27. Verification Summary

| Verification             | Expected  | Result |
| ------------------------ | --------- | ------ |
| R1 G0/0 up/up            | Yes       | PASS   |
| R1 G0/1 up/up            | Yes       | PASS   |
| R2 G0/0 up/up            | Yes       | PASS   |
| R2 G0/1 up/up            | Yes       | PASS   |
| R1 reaches R2 transit IP | Yes       | PASS   |
| R1 knows LAN A           | Connected | PASS   |
| R1 knows transit network | Connected | PASS   |
| R2 knows LAN B           | Connected | PASS   |
| R2 knows transit network | Connected | PASS   |
| R1 static route to LAN B | Present   | PASS   |
| R2 static route to LAN A | Present   | PASS   |
| PC0 → PC1                | Success   | PASS   |
| PC1 → PC0                | Success   | PASS   |

---

# 28. Final Architecture

```text
                 Transit Network
                  10.0.0.0/30

      10.0.0.1                  10.0.0.2
          G0/1                  G0/1
            R1 ================== R2
             │                    │
           G0/0                  G0/0
             │                    │
      192.168.10.1          192.168.20.1
             │                    │
            SW1                  SW2
             │                    │
            PC0                  PC1
       192.168.10.10       192.168.20.10
```

R1 has:

```text
C 192.168.10.0/24
C 10.0.0.0/30
S 192.168.20.0/24 via 10.0.0.2
```

R2 has:

```text
C 192.168.20.0/24
C 10.0.0.0/30
S 192.168.10.0/24 via 10.0.0.1
```

---

# 29. Final Learning Outcome

This practice lab demonstrated that routers do not automatically know every network in the topology.

A router automatically knows its directly connected networks:

```text
C = Connected
```

A remote network must be learned through another mechanism.

In this lab, that mechanism was:

```text
S = Static
```

The router was manually told:

```text
Remote network
      ↓
Next-hop router
```

This enabled end-to-end communication between two previously isolated LANs.

---

# 30. Final Takeaway

The most important concepts from this practice lab are:

```text
Directly Connected
        ↓
Router knows automatically

Remote Network
        ↓
Router needs a route

Static Route
        ↓
Manually specify destination + next hop

Two-way communication
        ↓
Both directions need a valid path
```

The core lesson is:

> **A router can reach a remote network only when its routing table contains a valid path to that network.**

Static routing is the simplest way to manually provide that path.

This practice lab provides the foundation for the next major routing topic:

```text
Static Routing
      ↓
Dynamic Routing Protocols
      ↓
OSPF
      ↓
EIGRP
      ↓
Larger Routed Networks
```
