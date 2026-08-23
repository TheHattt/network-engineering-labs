# MAC Learning Verification

## Initial MAC Address Table
The dynamic MAC Address table was checked before generating test traffic.
Command: 

```text
show mac address-table
```

The table was initially empty.

## Controlled MAC Address Learning Test

The dynamic MAC address table was cleared before the test was conducted.

The switch learned source MAC addresses from Ethernet frames received on its interfaces.

The resulting MAC Address table contained: 

| MAC Address | Port |
|-------------|------|
| 0002.4a12.cc11 | fast0/1 |
| 0009.7c78.9e9e | fast0/3 |


## Observations

The switch learned source MAC addresses from Ethernet frames received on its interfaces.

  - 0002.4a12.cc11 -> fast0/1
  - 0009.7c78.9e9e -> fast0/3

The entries were learned dynamically rather being manually configured as static MAC entries.


## Result

The controlled test confirmed the fundamental MAC address learning behavior.

```text
  Source MAC addresses -> Incoming switch port
```

--- 
## And now: 

This is our first evidence for MAC address learning.


```text
show mac address-table
```

Output:

| VLAN | MAC Address | Type | Port |
|------|-------------|------|------|
| 1 | 0002.4a12.cc11 | dynamic | fa0/1 |
| 1 | 0009.7c78.9e9e | dynamic | fa0/3 |

## Interpretation

The switch successfully learned the MAC addresses of the two PCs.
 - 0002.4a12.cc11 -> fast0/1
 - 0009.7c78.9e9e -> fast0/3

Both entries marked Dynamic, indicating that the switch learned these MAC addresses dynamically from received Ethernet traffic rather than from manual static configured entries.


## MAC Address Aging 

The Packet Tracer switch used in this lab does not support : 

```text
show mac address-table aging-time 
```

The command returned : 

```text
% Invalid input detected at '^' marker.
```

After the switch had been left inactive for an extended period, the dynamic MAC Address table was checked again.

```text
show mac address-table dynamic
```

The table no longer contained the previous learned dynamic entries.


## Result

The observation is consistent with dynamic MAC address aging: 

- Dynamically learned MAC Address entries can be removed from the MAC Address table after a period without relevant traffic.
- Therefore, this lab records the aging behavior as an observation rather than claiming a specific aging time value.


## ARP Broadcast Verification

### Test

PC0 initiated communicated with PC2:

```text
ping 192.168.10.12 
```


## ARP Reply and Known Unicast Forwarding Verification

### Test

PC0 initiated communication with PC2:

```text
ping 192.168.10.12 
```

The initial ARP request was broadcast by PC0 and flooded by the switch.
PC2 then generated an ARP reply.


## Observation

The ARP reply from PC2 was fowarded by the switch only toward PC0's interface.
The reply was not flooded through other active interfaces.



## Result

The simulation confirmed that the switch can foward a known-unicast frame through the specific interface associated with the destination MAC address.

```text
Destination MAC -> Learned switch port -> Forward
```



## Unknown Unicast Flooding Verification

### Initial Condition

The dynamic MAC Address table was empty before the test.

PC0 then initiated communication with PC2:


## Observation

The first ICMP frame was fowarded from the switch through the other active interface:
```text
Fa0/2, Fa0/3, Fa0/4
```

The destination MAC address was not yet known to the switch, so the frame was flooded within VLAN 1.

## MAC Learning After Traffic


After the traffic was processed, the switch's dynamic MAC address table contained:

| VLAN | MAC Address    | Type    | Switch Port |
|------|----------------|---------|-------------|
| 1    | 0002.4a12.cc11 |Dynamic  | Fa0/1 |
| 1    | 0009.7c7c.9e9e |Dynamic  | Fa0/3 |



## Result 

The simulation confirmed that when a switch receives a unicast frame whose destination MAC address is not present in its MAC address table, the switch floods the frame through the other active ports in the VLAN.

The switch also learns the source MAC address from the incoming frame.



## Known-Unicast Forwarding Verification

After the initial unknown-unicast frame was flooded, the switch learned the relevant MAC address information.

A subsequent ICMP frame from PC0 to PC2 was then forwarded only through `Fa0/3`.

### Observation

```text
PC0 → Switch → Fa0/3 → PC2


The frame was not forwarded through Fa0/2 or Fa0/4.


### Subsequent Frame — Known-Unicast Forwarding

After the switch learned the destination MAC address, the next ICMP frame from PC0 to PC2 was forwarded only through Fa0/3.

The switch no longer flooded the frame through the other active ports.

```text
PC0 → Switch0 → Fa0/3 → PC2
