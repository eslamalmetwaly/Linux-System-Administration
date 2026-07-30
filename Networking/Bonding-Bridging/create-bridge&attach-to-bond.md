# 01 - Create Linux Bridge

## Objective

Create a Linux Bridge named:

```text
br0
```

---

## Create Bridge

```bash
nmcli connection add type bridge \
con-name br0 \
ifname br0
```

---

## Verify

```bash
nmcli connection show
```

Expected:

```text
br0    bridge    br0
```

---

## Check Interface

```bash
ip link show br0
```

---

## Architecture

At this stage:

```text
br0
```

exists but does not yet have the Bond connected to it.

# 02 - Attach Bond to Linux Bridge

## Objective

Use the existing `bond0` interface as a Bridge Port.

Important:

```text
ens161 ───┐
          ├── bond0 ─── br0
ens224 ───┘
```

We do not attach the physical NICs directly to the Bridge.

The Bond is attached to the Bridge.

---

## Configure Bond as Bridge Port

```bash
nmcli connection modify bond0 \
connection.master br0 \
connection.slave-type bridge
```

---

## Verify

```bash
nmcli connection show bond0
```

Expected:

```text
connection.master: br0
connection.slave-type: bridge
```

---

## Check Bridge Ports

```bash
bridge link
```

Expected:

```text
bond0: master br0
```

---

## Final Architecture

```text
ens161 ───┐
          ├── bond0 ─── br0
ens224 ───┘
```

The Bond is now a Layer 2 port of the Linux Bridge.
# 03 - Configure IP on Linux Bridge

## Important Rule

The IP address must be configured on:

```text
br0
```

Not on:

```text
ens161
ens224
bond0
```

---

## Configure Static IP

Example:

```bash
nmcli connection modify br0 \
ipv4.method manual \
ipv4.addresses 192.168.43.50/24 \
ipv4.gateway 192.168.43.1 \
ipv4.dns "8.8.8.8 8.8.4.4"
```

---

## Activate Bridge

```bash
nmcli connection up br0
```

---

## Verify IP

```bash
ip addr show br0
```

---

## Verify Route

```bash
ip route
```

Expected:

```text
default via 192.168.43.1 dev br0
```

---

## Correct Layer 3 Design

```text
IP Address
    │
   br0
    │
  bond0
    │
ens161 / ens224
```
