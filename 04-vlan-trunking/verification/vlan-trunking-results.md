# Lab 04 — VLAN Trunking Verification Results

## Purpose

This document records the verification results for the VLAN trunking lab.

The objective was to verify that:

- SW1 and SW2 are connected through an IEEE 802.1Q trunk.
- VLAN 10 is transported across the trunk.
- VLAN 20 is transported across the trunk.
- Same-VLAN communication works across the two switches.
- Inter-VLAN communication remains isolated.
- MAC addresses from remote VLANs are learned through the trunk.
- The trunk is operational on both switches.

---

# 1. Pre-Trunk Connectivity Test

Before configuring the inter-switch link as a trunk, the switches did not have a functioning Layer 2 path for VLAN 10 and VLAN 20 traffic.

### VLAN 20 Test

![Pre-trunk VLAN 20 failure](../screenshots/08-pre-trunk-vlan20-failure.png)

The VLAN 20 endpoint could not reach the corresponding VLAN 20 endpoint on the other switch.

**Result:** FAIL — expected before trunk configuration.

### VLAN 10 Test

![Pre-trunk VLAN 10 failure](../screenshots/09-pre-trunk-vlan10-failure.png)

The VLAN 10 endpoint could not reach the corresponding VLAN 10 endpoint on the other switch.

**Result:** FAIL — expected before trunk configuration.

These failures established the baseline condition before the trunk was configured.

---

# 2. Trunk Configuration Verification

The inter-switch connection uses:

```text
SW1 Fa0/24 <----------> SW2 Fa0/24
