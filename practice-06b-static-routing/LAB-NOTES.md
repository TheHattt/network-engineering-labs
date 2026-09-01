
# Practice Lab 06B — Static Routing Between Two LANs

## Lab Notes

---

# 1. Lab Overview

This practice lab introduces **static routing** between two separate LANs connected through two routers.

The lab builds directly on the Layer 3 concepts introduced in Lab 06.

Lab 06 demonstrated that a multilayer switch can route between networks that are directly connected through its SVIs.

This practice lab introduces a different situation:

```text
The destination network is NOT directly connected to the router.
```

That creates the need for an explicit route.

The progression is:

```text
Directly Connected Networks
        ↓
Remote Networks
        ↓
Static Routes
        ↓
End-to-End Routing
```

The purpose of this exercise is to understand how a router learns about a remote network and how the forwarding path changes after a static route is installed.

---

# 2. The Problem This Lab Solves

Consider the following topology:

```text
PC0 ─ SW1 ─ R1 ───────── R2 ─ SW2 ─ PC1
              LAN A       LAN B
```

There are three IP networks:

```text
LAN A:
192.168.10.0/24

Transit:
10.0.0.0/30

LAN B:
192.168.20.0/24
```

R1 is directly connected to:

```text
192.168.10.0/24
10.0.0.0/30
```

R2 is directly connected to:

```text
10.0.0.0/30
192.168.20.0/24
```

The problem is that R1 does not automatically know about:

```text
192.168.20.0/24
```

and R2 does not automatically know about:

```text
192.168.10.0/24
```

The routers therefore need additional routing information.

---

# 3. Topology

The topology consists of:

```text
2 PCs
2 Layer 2 switches
2 routers
```

The physical relationships are:

```text
PC0
 │
SW1 Fa0/24
 │
R1 G0/0
R1 G0/1
 │
 │ 10.0.0.0/30
 │
R2 G0/1
R2 G0/0
 │
SW2 Fa0/24
 │
PC1
```

The routing architecture is:

```text
LAN A
192.168.10.0/24
       │
      R1
       │
Transit Network
10.0.0.0/30
       │
      R2
       │
LAN B
192.168.20.0/24
```

---

# 4. Interface Layout

The actual physical interface assignment is important because the interface numbers determine where the IP addresses belong.

## R1

```text
G0/0 → SW1 / LAN A
G0/1 → R2 / Transit
```

## R2

```text
G0/0 → SW2 / LAN B
G0/1 → R1 / Transit
```

Therefore the IP addressing is:

```text
R1 G0/0 → 192.168.10.1/24
R1 G0/1 → 10.0.0.1/30

R2 G0/1 → 10.0.0.2/30
R2 G0/0 → 192.168.20.1/24
```

---

# 5. Endpoint Addressing

## PC0

```text
IP Address:
192.168.10.10

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.10.1
```

## PC1

```text
IP Address:
192.168.20.10

Subnet Mask:
255.255.255.0

Default Gateway:
192.168.20.1
```

The hosts are therefore located in different IP networks.

---

# 6. The Transit Network

The two routers communicate over:

```text
10.0.0.0/30
```

A `/30` subnet provides:

```text
Network:
10.0.0.0

Usable Host:
10.0.0.1

Usable Host:
10.0.0.2

Broadcast:
10.0.0.3
```

The addressing is therefore:

```text
R1 G0/1 → 10.0.0.1
R2 G0/1 → 10.0.0.2
```

The transit network exists only to connect the routers.

It is not the end-user LAN.

---

# 7. What "Transit Network" Means

A transit network is simply:

> A network used to carry traffic between network devices.

In this topology:

```text
LAN A
   │
  R1
   │
   │ Transit
   │
  R2
   │
LAN B
```

The transit network is the path between R1 and R2.

It allows R1 to reach R2 and R2 to reach R1.

---

# 8. Directly Connected Routes

After configuring the interfaces, R1 automatically learns:

```text
C 192.168.10.0/24
C 10.0.0.0/30
```

because those networks are directly connected to R1.

Likewise, R2 automatically learns:

```text
C 10.0.0.0/30
C 192.168.20.0/24
```

because those networks are directly connected to R2.

The important distinction is:

```text
R1 knows its local LAN.
R2 knows its local LAN.
```

But:

```text
R1 does not know LAN B.
R2 does not know LAN A.
```

---

# 9. Why Direct Connectivity Matters

A router does not need a manually configured route for a network physically attached to one of its interfaces.

For example:

```text
R1 G0/0
192.168.10.1/24
```

automatically causes R1 to know:

```text
192.168.10.0/24
```

This becomes a:

```text
C = Connected
```

route.

The router learns this from the interface configuration.

---

# 10. The Initial Routing Problem

PC0 attempted to reach:

```text
192.168.20.10
```

The destination belongs to:

```text
192.168.20.0/24
```

PC0 therefore sent the traffic to its default gateway:

```text
192.168.10.1
```

R1 received the traffic.

The problem was that R1's routing table did not contain:

```text
192.168.20.0/24
```

Therefore R1 did not know where to send the traffic.

The packet could not complete the journey.

---

# 11. Why the Transit Link Being Up Was Not Enough

R1 successfully reached:

```text
10.0.0.2
```

using:

```text
ping 10.0.0.2
```

This proved:

```text
R1 ↔ R2
```

was working.

However, this did not automatically mean:

```text
R1 → 192.168.20.0/24
```

would work.

The important distinction is:

```text
Reachability to R2
        ≠
Knowledge of R2's LAN
```

R1 knew how to reach the router itself.

It did not yet know that R2 was the path to the remote LAN.

---

# 12. Static Routing Concept

A static route manually tells a router:

```text
Destination Network
        ↓
Next-Hop Router
```

For R1:

```text
Destination:
192.168.20.0/24

Next Hop:
10.0.0.2
```

The meaning is:

> To reach the 192.168.20.0/24 network, send the traffic to R2 at 10.0.0.2.

---

# 13. Static Route on R1

The resulting routing table entry is:

```text
S 192.168.20.0/24 [1/0] via 10.0.0.2
```

The important pieces are:

```text
S
```

which means:

```text
Static
```

and:

```text
192.168.20.0/24
```

which is the destination network.

and:

```text
via 10.0.0.2
```

which is the next-hop router.

The entire entry can be read as:

> R1 reaches 192.168.20.0/24 through R2 at 10.0.0.2.

---

# 14. Static Route on R2

R2 needs the reverse path.

Its remote network is:

```text
192.168.10.0/24
```

and the next-hop router is:

```text
10.0.0.1
```

The resulting route is:

```text
S 192.168.10.0/24 [1/0] via 10.0.0.1
```

This tells R2:

> To reach 192.168.10.0/24, send the traffic to R1 at 10.0.0.1.

---

# 15. Why Both Static Routes Are Needed

It is not enough for only R1 to know how to reach PC1.

Consider:

```text
PC0 → PC1
```

The packet travels:

```text
PC0
 ↓
R1
 ↓
R2
 ↓
PC1
```

PC1 then generates a reply.

The reply must travel:

```text
PC1
 ↓
R2
 ↓
R1
 ↓
PC0
```

Therefore R2 must know how to reach:

```text
192.168.10.0/24
```

This is the concept of a **return path**.

---

# 16. Forward Path

After installing the route on R1:

```text
PC0
192.168.10.10
   ↓
Default Gateway
192.168.10.1
   ↓
R1
   ↓
Static Route
192.168.20.0/24
via 10.0.0.2
   ↓
R2
   ↓
192.168.20.0/24
   ↓
PC1
192.168.20.10
```

The forward direction is now known.

---

# 17. Return Path

After installing the route on R2:

```text
PC1
192.168.20.10
   ↓
Default Gateway
192.168.20.1
   ↓
R2
   ↓
Static Route
192.168.10.0/24
via 10.0.0.1
   ↓
R1
   ↓
192.168.10.0/24
   ↓
PC0
192.168.10.10
```

Both directions are now known.

---

# 18. Routing Table Evolution

The most useful learning point in this lab is watching the routing tables change.

## R1 before the static route

```text
C 192.168.10.0/24
C 10.0.0.0/30
```

Remote LAN:

```text
192.168.20.0/24
```

was missing.

## R1 after the static route

```text
C 192.168.10.0/24
C 10.0.0.0/30
S 192.168.20.0/24 via 10.0.0.2
```

R1 can now reach LAN B.

---

# 19. R2 Before the Static Route

R2 initially knew:

```text
C 10.0.0.0/30
C 192.168.20.0/24
```

The remote LAN:

```text
192.168.10.0/24
```

was missing.

After adding the static route:

```text
C 10.0.0.0/30
C 192.168.20.0/24
S 192.168.10.0/24 via 10.0.0.1
```

R2 can now reach LAN A.

---

# 20. Understanding the `[1/0]`

A static route is displayed as:

```text
S 192.168.20.0/24 [1/0] via 10.0.0.2
```

The brackets represent:

```text
[Administrative Distance / Metric]
```

For the default static route values:

```text
Administrative Distance = 1
Metric = 0
```

For now, the important part is recognizing the route type:

```text
C = Connected
S = Static
```

Later, this becomes important when comparing multiple routing sources.

---

# 21. Connectivity Before and After Routing

## Before static routing

```text
PC0 → PC1
```

failed because R1 had no route to LAN B.

## After R1 route only

The packet could be forwarded toward R2, but the complete communication still failed because R2 did not yet have the return route.

## After both static routes

```text
PC0 → PC1
```

succeeded.

And:

```text
PC1 → PC0
```

also succeeded.

This demonstrated the need for a complete two-way routing path.

---

# 22. Important Troubleshooting Observation

The error changed after R1 received its static route.

Before the route:

```text
Destination host unreachable
```

was observed.

After the R1 route was added:

```text
Request timed out
```

was observed.

This indicated that the packet had progressed farther into the topology.

The first problem was:

```text
R1 does not know the remote network.
```

After fixing that, the remaining problem became:

```text
R2 does not know the return network.
```

This is a useful troubleshooting lesson:

> A change in the error can indicate that the packet has progressed farther through the network.

---

# 23. Static Routing vs Dynamic Routing

Static routing requires manual configuration.

For example:

```text
R1
192.168.20.0/24
via 10.0.0.2
```

A dynamic routing protocol would allow routers to exchange route information automatically.

Conceptually:

```text
Static Routing
        ↓
Manual route configuration
```

versus:

```text
Dynamic Routing
        ↓
Routers exchange route information
```

Dynamic routing will become important in later labs.

---

# 24. Why This Is Different From Lab 06

Lab 06 dealt primarily with:

```text
Directly connected networks
```

The multilayer switch automatically knew its VLAN networks through the SVIs.

This practice lab introduces:

```text
Remote networks
```

R1 cannot automatically know about LAN B.

R2 cannot automatically know about LAN A.

The missing information must therefore be provided.

The first solution is:

```text
Static routing
```

---

# 25. Layer 3 Mental Model

The routing process can be remembered as:

```text
Destination IP
      ↓
Check local network
      ↓
If remote
      ↓
Send to default gateway
      ↓
Router checks routing table
      ↓
Find destination network
      ↓
Select next hop
      ↓
Forward
```

In this lab:

```text
PC0
 ↓
R1
 ↓
Static route
 ↓
R2
 ↓
PC1
```

---

# 26. Troubleshooting Workflow

When PC0 cannot reach PC1, follow this sequence:

```text
1. Check PC0 IP and gateway
        ↓
2. Check R1 interfaces
        ↓
3. Check R1 ↔ R2 connectivity
        ↓
4. Check R1 routing table
        ↓
5. Check R1 route to LAN B
        ↓
6. Check R2 routing table
        ↓
7. Check R2 route to LAN A
        ↓
8. Check PC1 gateway
        ↓
9. Test again
```

This prevents random configuration changes.

---

# 27. Commands Used

## Router interfaces

```cisco
show ip interface brief
```

Purpose:

Verify IP addresses and interface operational status.

## Routing table

```cisco
show ip route
```

Purpose:

Identify connected and static routes.

## Connectivity

```cisco
ping <destination>
```

Purpose:

Verify reachability between nodes.

## Configuration

```cisco
show running-config
```

Purpose:

Inspect the actual router configuration.

---

# 28. Key Configuration Concepts

### Connected route

```text
C
```

A network directly connected to a router interface.

### Static route

```text
S
```

A route manually entered by the administrator.

### Next hop

The router to which traffic should be forwarded next.

### Transit network

The network connecting routers.

### Remote network

A network not directly connected to the current router.

### Return path

The path used by reply traffic to get back to the original source.

---

# 29. Interview Questions

### Why did R1 automatically know 192.168.10.0/24?

Because the network was directly connected to R1's G0/0 interface.

### Why didn't R1 automatically know 192.168.20.0/24?

Because that network exists behind R2 and is not directly connected to R1.

### What is a static route?

A manually configured route that tells a router how to reach a destination network.

### What is a next hop?

The next router to which traffic should be forwarded on the way to its destination.

### Why are two static routes required?

Because bidirectional communication requires both a forward path and a return path.

### What does `C` mean in the routing table?

Connected.

### What does `S` mean?

Static.

### What is a transit network?

A network used to connect routers or other network infrastructure.

### Why did the transit ping work while PC0 to PC1 failed?

The routers could reach each other, but R1 initially had no route to the remote LAN.

---

# 30. Final Mental Model

The complete process is:

```text
PC0
192.168.10.10
    ↓
Default Gateway
192.168.10.1
    ↓
R1
    ↓
Routing Table
    ↓
Static Route
192.168.20.0/24 via 10.0.0.2
    ↓
Transit Network
10.0.0.0/30
    ↓
R2
    ↓
Directly Connected Network
192.168.20.0/24
    ↓
PC1
192.168.20.10
```

The reply travels in the opposite direction using R2's static route.

---

# 31. Final Learning Outcome

This practice lab establishes the relationship between:

```text
Directly Connected Networks
        ↓
Connected Routes

Remote Networks
        ↓
Static Routes

Two-Way Communication
        ↓
Forward + Return Routes
```

The key lesson is:

> **A router knows its directly connected networks automatically, but remote networks require routing information.**

In this lab, that information was manually provided using static routes.

---

# 32. Final Takeaway

The most important mental model from this practice lab is:

```text
LOCAL NETWORK
      ↓
DEFAULT GATEWAY
      ↓
ROUTER
      ↓
ROUTING TABLE
      ↓
STATIC ROUTE
      ↓
NEXT HOP
      ↓
TRANSIT NETWORK
      ↓
REMOTE ROUTER
      ↓
REMOTE NETWORK
```

The two routers must have enough information for traffic to travel in both directions.

That is the foundation of static routing and the bridge into dynamic routing protocols.

---

# 33. Progression to the Next Stage

The routing concepts now build naturally:

```text
Lab 06
Multilayer Switching
        ↓
SVIs + Connected Routes
        ↓
Practice 06B
Static Routing
        ↓
Remote Networks
        ↓
Next-Hop Routing
        ↓
Dynamic Routing
        ↓
OSPF / EIGRP
```

This practice lab therefore provides the first hands-on foundation for understanding how larger routed networks learn and maintain paths between remote networks.
