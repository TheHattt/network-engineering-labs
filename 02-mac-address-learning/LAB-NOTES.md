#Lab Notes 

## ARP Broadcast Observation

PC0 initiated communication with PC2 (`192.168.10.12`).

The first communication required ARP because pc0 needed to determine the MAC address associated with PC2's (`192.168.10.12`) IP address.

The ARP request was into the local VLAN.

The switch received the frame on fa0/1 and flooded the broadcast out the active ports in VLAN 1.


### Observed behavior

 - The switch learned the source MAC address from the incoming frame.
 - The ARP request was flooded because it was a broadcast.
 - The Incoming port was not used as an output flood port.
 - Devices on the same VLAN received the broadcast.


 ## ARP Reply and Known-Unicast Forwarding

 After the switch flooded the ARP request, PC2 generated an ARP reply.
 The ARP reply was received by th switch and fowarded only through the the interface connected to PC0.

 The Reply was not flooded to the other PCs in the VLAN.


 ## Observation

 ```text
  PC2 -> Switch -> Fa0/1 -> PC0
 ```
The switch was able to foward the reply toward PC0 because it had learned PC0's source MAC address on Fa0/1.

This demonstrates the difference between broadcast flooding and known unicast forwarding.



## Unknown Unicast Flooding

The dynamic MAC address table was empty before the test.
PC0 initiated communication with PC2.

The first ICMP frame was fowarded through the other active ports because the destination MAC address was not yet present in the switch's MAC address table.

The switch flooded the frame through:

```text
Fa0/2, Fa0/3, Fa0/4
```



The switch simultaneously learned source MAC information from received Ethernet traffic.

After the traffic was processed, the MAC address table contained dynamically learned entries.

This demonstrated that an unknown unicast destination is flooded within the VLAN until the switch learns the destination MAC address.


## Known Unicast Fowarding After MAC Address Learning

After the switch learned the destination MAC Address, subsequent ICMP traffic from PC0 to PC2 no longer flooded.

The next ICMP frame was fowarded only through:

```text
Fa0/3
           
The switch used the learned destination MAC address to determine the correct outgoing interface.

PC0 -> Switch -> Fa0/3 -> PC2

This demonstrated the transition from unknown unicast flooding to known unicast forwarding.
