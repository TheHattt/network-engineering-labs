# `show interface fa0/3` Verification

## After `shutdown`

```text
FastEthernet0/3 is administratively down, line protocol is down (disabled)
```

Relevant counters observed:

```text
0 input errors
0 CRC
0 frame
0 overrun
0 ignored
0 output errors
0 collisions
```

## After `no shutdown`

```text
FastEthernet0/3 is up, line protocol is up (connected)
```

Recovery messages observed:

```text
%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/3, changed state to up
```
