# Ping Verification

## Test 1 — Normal operation

Source: PC0

Destination: `192.168.10.12`

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

## Test 2 — Fa0/3 shutdown

Fa0/3 was administratively disabled.

```text
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

## Test 3 — Fa0/3 restored

Fa0/3 was enabled with `no shutdown`.

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

## Conclusion

The results demonstrate the relationship between the operational state of the switch interface and end-to-end host connectivity.
