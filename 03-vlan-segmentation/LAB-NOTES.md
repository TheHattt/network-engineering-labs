# Lab 03 — VLAN Segmentation and Layer 2 Broadcast Domains

## Learning Objective

Understand how VLANs divide a physical switch into separate logical Layer 2 broadcast domains.

This lab builds directly on Lab 02, where MAC address learning, ARP, unknown-unicast flooding, and known-unicast forwarding were examined.

The key question for this lab is:

> How can one physical switch provide multiple isolated Layer 2 networks?

---

# 1. VLAN Fundamentals

VLAN stands for:

**Virtual Local Area Network**

A VLAN is a logical Layer 2 segmentation mechanism.

Instead of placing every connected device into one Layer 2 network, a switch can divide its ports into different VLANs.

Example:

```text
                 SW1
                  |
        +---------+---------+
        |                   |
     VLAN 10             VLAN 20
      USERS              FINANCE
     /     \              /    \
   PC0     PC1          PC2    PC3
