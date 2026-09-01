
# Practice Lab 06B — Static Routing Between Two LANs

## Verification Results

---

# 1. Verification Overview

This document records the verification performed on the two-router static-routing topology.

The purpose of the verification was to prove that two separate LANs can communicate through an intermediate transit network after the routers are manually configured with routes to their respective remote networks.

The verification demonstrates:

```text
Directly Connected Networks
        ↓
Transit Connectivity
        ↓
Remote Networks
        ↓
Static Routes
        ↓
Forward Path + Return Path
        ↓
End-to-End Communication
```

The final topology successfully supports bidirectional communication between PC0 and PC1.

---

# 2. Topology Under Test

The topology is:

```text
PC0 ── SW1 ── R1 ───────── R2 ── SW2 ── PC1
                 │          │
                 └─ Transit ┘
```

The three networks are:

```text
LAN A:
192.168.10.0/24

Transit:
10.0.0.0/30

LAN B:
192.168.20.0/24
```

The routers use the transit network to communicate with each other.

---

# 3. Device Addressing

| Device | Interface | Address            | Network           | Role          |
| ------ | --------- | ------------------ | ----------------- | ------------- |
| PC0    | NIC       | `192.168.10.10/24` | `192.168.10.0/24` | LAN A host    |
| R1     | G0/0      | `192.168.10.1/24`  | `192.168.10.0/24` | LAN A gateway |
| R1     | G0/1      | `10.0.0.1/30`      | `10.0.0.0/30`     | Transit       |
| R2     | G0/1      | `10.0.0.2/30`      | `10.0.0.0/30`     | Transit       |
| R2     | G0/0      | `192.168.20.1/24`  | `192.168.20.0/24` | LAN B gateway |
| PC1    | NIC       | `192.168.20.10/24` | `192.168.20.0/24` | LAN B host    |

---

# 4. Interface Status Verification

## R1

Command:

```cisco
show ip interface brief
```

R1 was verified with:

```text
GigabitEthernet0/0     192.168.10.1    up    up
GigabitEthernet0/1     10.0.0.1        up    up
```

### Result

**PASS**

Both R1 interfaces are operational.

---

## R2

Command:

```cisco
show ip interface brief
```

R2 was verified with:

```text
GigabitEthernet0/0     192.168.20.1    up    up
GigabitEthernet0/1     10.0.0.2        up    up
```

### Result

**PASS**

Both R2 interfaces are operational.

---

# 5. R1 Initial Routing Table

Before static routing was introduced, R1 was checked with:

```cisco
show ip route
```

R1 knew:

```text
C 10.0.0.0/30
C 192.168.10.0/24
```

and their corresponding local routes.

R1 did **not** know:

```text
192.168.20.0/24
```

### Result

**EXPECTED**

R1 knew the networks directly connected to itself but had no information about the remote LAN behind R2.

---

# 6. R2 Initial Routing Table

R2 was also checked before static routing.

R2 knew:

```text
C 10.0.0.0/30
C 192.168.20.0/24
```

R2 did not know:

```text
192.168.10.0/24
```

### Result

**EXPECTED**

R2 knew its directly connected networks but did not yet know how to reach the remote LAN behind R1.

---

# 7. Directly Connected Network Verification

The routers' initial routing knowledge can be summarized as:

```text
R1:
192.168.10.0/24 → Connected
10.0.0.0/30     → Connected
192.168.20.0/24 → Unknown

R2:
192.168.20.0/24 → Connected
10.0.0.0/30     → Connected
192.168.10.0/24 → Unknown
```

This demonstrates the difference between:

```text
Local / directly connected network
```

and:

```text
Remote network
```

A router learns a directly connected network automatically from its interface configuration.

It does not automatically know a remote LAN simply because another router is connected.

---

# 8. Transit Link Verification

The R1-to-R2 transit connection uses:

```text
10.0.0.0/30
```

with:

```text
R1 → 10.0.0.1
R2 → 10.0.0.2
```

The link was tested from R1:

```cisco
ping 10.0.0.2
```

The result was:

```text
Success rate is 80 percent (4/5)
```

The first probe did not receive a reply, while the following probes succeeded.

### Result

**PASS**

R1 can communicate with R2 across the transit network.

### What this proves

The routers have basic Layer 3 reachability across:

```text
10.0.0.0/30
```

This is important because the transit link must work before it can be used as the next-hop path for remote networks.

---

# 9. Why the Transit Link Alone Was Not Enough

Even though R1 could reach:

```text
10.0.0.2
```

it still could not reach:

```text
192.168.20.0/24
```

This demonstrates:

```text
Reach R2
    ≠
Know how to reach R2's LAN
```

The router needed an explicit routing entry telling it that R2 is the path to the remote LAN.

---

# 10. Initial End-to-End Test

PC0 attempted to reach PC1:

```text
PC0:
192.168.10.10

Destination:
192.168.20.10
```

Before static routing was configured, the test failed with a destination-unreachable response from the local gateway.

This was expected.

### Cause

R1 did not have:

```text
192.168.20.0/24
```

in its routing table.

Therefore R1 had no route for the destination network.

### Result

**EXPECTED FAILURE**

The failure confirmed that the topology was ready for the static-routing exercise.

---

# 11. Static Route Added to R1

R1 was given a route to the remote LAN:

```text
Destination:
192.168.20.0/24

Next hop:
10.0.0.2
```

The resulting routing-table entry became:

```text
S 192.168.20.0/24 [1/0] via 10.0.0.2
```

### Result

**PASS**

R1 now has a valid path to LAN B.

### Interpretation

The entry means:

> To reach the 192.168.20.0/24 network, forward the traffic to R2 at 10.0.0.2.

---

# 12. Routing Table After R1 Static Route

R1's routing knowledge is now:

```text
C 192.168.10.0/24
C 10.0.0.0/30
S 192.168.20.0/24 via 10.0.0.2
```

The routing table has changed from:

```text
Remote network:
Unknown
```

to:

```text
Remote network:
Known via static route
```

This is the direct effect of the static route.

---

# 13. End-to-End Test After R1 Route

After adding the R1 route, PC0 tested:

```text
ping 192.168.20.10
```

The response changed from:

```text
Destination host unreachable
```

to:

```text
Request timed out
```

### Result

**PARTIAL PROGRESS**

This was an important troubleshooting observation.

The original problem had been reduced: R1 now knew how to reach the remote network.

However, the complete communication path was still missing.

---

# 14. Why the Ping Still Failed

R1 could now send traffic toward:

```text
R2 → 192.168.20.0/24
```

But successful IP communication requires a return path.

PC1's reply must travel back toward:

```text
192.168.10.0/24
```

R2 initially did not have that route.

R2 knew:

```text
192.168.20.0/24
10.0.0.0/30
```

but did not know:

```text
192.168.10.0/24
```

Therefore the forward path existed, but the return path did not.

---

# 15. Static Route Added to R2

R2 was then configured with the reverse route:

```text
Destination:
192.168.10.0/24

Next hop:
10.0.0.1
```

The resulting routing-table entry became:

```text
S 192.168.10.0/24 [1/0] via 10.0.0.1
```

### Result

**PASS**

R2 now has a valid path back toward LAN A.

---

# 16. Final R1 Routing Knowledge

R1 now knows:

```text
C 192.168.10.0/24
C 10.0.0.0/30
S 192.168.20.0/24 via 10.0.0.2
```

This means:

```text
Local LAN:
192.168.10.0/24

Transit:
10.0.0.0/30

Remote LAN:
192.168.20.0/24
        ↓
    via R2
```

---

# 17. Final R2 Routing Knowledge

R2 now knows:

```text
C 192.168.20.0/24
C 10.0.0.0/30
S 192.168.10.0/24 via 10.0.0.1
```

This means:

```text
Local LAN:
192.168.20.0/24

Transit:
10.0.0.0/30

Remote LAN:
192.168.10.0/24
        ↓
    via R1
```

---

# 18. Forward Path Verification

The final forward path is:

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
Transit Network
10.0.0.0/30
    ↓
R2
    ↓
Directly Connected
192.168.20.0/24
    ↓
PC1
192.168.20.10
```

### Result

**PASS**

A valid forward path exists.

---

# 19. Return Path Verification

The reply follows:

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
Transit Network
10.0.0.0/30
    ↓
R1
    ↓
Directly Connected
192.168.10.0/24
    ↓
PC0
192.168.10.10
```

### Result

**PASS**

A valid return path exists.

---

# 20. Final End-to-End Test — PC0 to PC1

After both static routes were configured:

```text
PC0 → 192.168.20.10
```

succeeded.

### Result

**PASS**

This proves:

```text
LAN A
   ↓
R1
   ↓
Transit Network
   ↓
R2
   ↓
LAN B
```

is reachable end-to-end.

---

# 21. Final End-to-End Test — PC1 to PC0

The reverse test:

```text
PC1 → 192.168.10.10
```

also succeeded.

### Result

**PASS**

This confirms that the routing design works in both directions.

---

# 22. Why the Return Route Was Necessary

This lab demonstrated an important routing principle:

> A packet reaching its destination does not guarantee successful communication.

The destination must also be able to return the response.

Therefore:

```text
Forward route
+
Return route
=
Two-way communication
```

Without the R2 static route, the request could reach the remote LAN but the response had no valid path back.

---

# 23. Connected vs Static Route Verification

The final routing tables contain both:

```text
C
```

and:

```text
S
```

### Connected route

```text
C 192.168.10.0/24
```

means:

> This network is directly connected to the router.

### Static route

```text
S 192.168.20.0/24 via 10.0.0.2
```

means:

> This network is remote and was manually configured through the specified next hop.

The practical distinction is:

```text
C → Router discovers automatically
S → Administrator configures manually
```

---

# 24. Verification Matrix

| Test                 | Expected Result | Actual Result |
| -------------------- | --------------- | ------------- |
| R1 G0/0 up/up        | Yes             | PASS          |
| R1 G0/1 up/up        | Yes             | PASS          |
| R2 G0/0 up/up        | Yes             | PASS          |
| R2 G0/1 up/up        | Yes             | PASS          |
| R1 → R2 transit ping | Success         | PASS          |
| R1 knows LAN A       | Connected       | PASS          |
| R1 knows transit     | Connected       | PASS          |
| R2 knows LAN B       | Connected       | PASS          |
| R2 knows transit     | Connected       | PASS          |
| R1 knows LAN B       | Static          | PASS          |
| R2 knows LAN A       | Static          | PASS          |
| PC0 → PC1            | Success         | PASS          |
| PC1 → PC0            | Success         | PASS          |
| Two-way routing      | Success         | PASS          |

---

# 25. Troubleshooting Summary

The troubleshooting sequence can be summarized as:

```text
PC0 → PC1 failed
        ↓
Check R1
        ↓
Remote network missing
        ↓
Add static route to R1
        ↓
Packet can move toward R2
        ↓
End-to-end still fails
        ↓
Check R2
        ↓
Return route missing
        ↓
Add static route to R2
        ↓
PC0 ↔ PC1 succeeds
```

This progression demonstrated how routing problems can be isolated by inspecting the routing tables.

---

# 26. Key Observations

### Observation 1

A router automatically learns directly connected networks.

### Observation 2

A remote network does not automatically appear in the routing table.

### Observation 3

A transit link provides router-to-router reachability, but does not automatically advertise the remote LAN.

### Observation 4

A static route tells a router where a remote network can be found.

### Observation 5

Two-way communication requires both a forward path and a return path.

### Observation 6

Changes in ping behavior can help determine how far traffic is progressing through the network.

---

# 27. Final Routing Diagram

```text
                      TRANSIT
                    10.0.0.0/30
             10.0.0.1      10.0.0.2

PC0 ─ SW1 ─── R1 ═══════════ R2 ─── SW2 ─ PC1
        │                       │
        │                       │
192.168.10.0/24          192.168.20.0/24
        │                       │
       LAN A                   LAN B
```

R1:

```text
192.168.10.0/24 → Connected
10.0.0.0/30     → Connected
192.168.20.0/24 → Static via 10.0.0.2
```

R2:

```text
192.168.20.0/24 → Connected
10.0.0.0/30     → Connected
192.168.10.0/24 → Static via 10.0.0.1
```

---

# 28. Final Result

**PRACTICE LAB 06B — PASSED**

The two-router topology successfully provides bidirectional connectivity between:

```text
192.168.10.0/24
```

and:

```text
192.168.20.0/24
```

through:

```text
10.0.0.0/30
```

The final routing design consists of:

```text
Connected routes
+
Static route on R1
+
Static route on R2
+
Correct host gateways
```

The final end-to-end tests succeeded in both directions.

---

# 29. Final Learning Outcome

This practice lab established the foundation for understanding remote routing.

The key progression was:

```text
Directly Connected
        ↓
Router automatically knows
        ↓
Remote Network
        ↓
No route exists
        ↓
Static Route
        ↓
Next Hop
        ↓
Remote Network becomes reachable
```

For bidirectional communication:

```text
Forward Route
+
Return Route
        ↓
Successful communication
```

The most important takeaway is:

> **A router can communicate with a remote network only when its routing table contains a valid path to that network.**

In this exercise, that path was manually provided with static routes.
