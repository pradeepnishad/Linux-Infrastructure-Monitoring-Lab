# Troubleshooting 06: Temporary IP Addresses and Persistent Networking

## Problem

The private IPv4 addresses configured on the VMs were missing during later tests. This caused application connectivity and reverse proxy requests to fail.

## Root Cause

The addresses were assigned using temporary commands such as:

```bash
sudo ip addr add 10.10.10.12/24 dev enp0s8
```

Addresses configured this way do not persist across a reboot.

## Temporary Recovery Commands

On `web01`:

```bash
sudo ip addr add 10.10.10.11/24 dev enp0s8
sudo ip link set enp0s8 up
```

On `app01`:

```bash
sudo ip addr add 10.10.10.12/24 dev enp0s8
sudo ip link set enp0s8 up
```

On `db01`:

```bash
sudo ip addr add 10.10.10.13/24 dev enp0s8
sudo ip link set enp0s8 up
```

Only run these commands after checking the current addresses. If an address is already configured, do not add it repeatedly.

## Verification

On each VM:

```bash
ip -br addr
ip route
```

From `web01`:

```bash
ping -c 4 10.10.10.12
curl -i http://10.10.10.12:8000/health
```

From `app01`:

```bash
ping -c 4 10.10.10.13
nc -vz 10.10.10.13 5432
```

## Permanent Fix: Netplan

The next required improvement is to configure the internal IPv4 addresses persistently using Netplan.

Before changing network configuration:

1. Inspect the existing files under `/etc/netplan/`.
2. Identify the correct interface names with `ip -br link`.
3. Preserve the existing NAT/DHCP configuration.
4. Configure the internal interface with the appropriate static address.
5. Validate the YAML and test connectivity carefully.

Do not blindly replace the existing Netplan configuration, because doing so could remove NAT connectivity or disconnect SSH.

## Key Learning

Reliable infrastructure requires persistent network configuration. A manually assigned IP can fix an immediate issue, but persistent configuration prevents the same issue from recurring after a reboot.
