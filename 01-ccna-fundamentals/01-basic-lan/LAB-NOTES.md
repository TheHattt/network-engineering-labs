# Lab Notes

## Findings

A switch interface can be administratively disabled and later restored without changing the IP configuration of the connected hosts.

`shutdown` disables the interface administratively.

`no shutdown` removes the administrative shutdown.

When the interface recovers, Cisco reports the physical link state and then the line protocol state.

## Troubleshooting distinction

`administratively down` identifies an interface that has been disabled by configuration.

A plain `down` state should not automatically be interpreted as an administrative shutdown. Physical connectivity, the remote device, and the cable should be checked.

## Evidence from this lab

```text
up/up
administratively down/down
up/up
```

Corresponding connectivity results:

```text
0% loss
100% loss
0% loss
```

## Documentation rule

Only record IP addresses, interface mappings, device names, and results that were actually observed in the Packet Tracer topology or command output.
