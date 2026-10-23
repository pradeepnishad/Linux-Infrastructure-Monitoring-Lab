# Troubleshooting 04: Missing Private IPv4 Address on app01

## Problem

FastAPI responded successfully on `app01`:

```bash
curl -i http://127.0.0.1:8000/health
```

However, connections from `web01` to `10.10.10.12:8000` timed out.

## Environment

- Application VM: `app01`
- Internal interface: `enp0s8`
- Expected private IP: `10.10.10.12/24`
- FastAPI port: `8000`

## Investigation

Inspect network interfaces:

```bash
ip -br addr
```

Inspect listening ports:

```bash
sudo ss -lntp | grep 8000
```

The application was listening on:

```text
0.0.0.0:8000
```

But `enp0s8` had only an IPv6 address, with no private IPv4 address.

## Root Cause

The private IPv4 address had been configured temporarily and was no longer present.

## Resolution

Restored the IPv4 address:

```bash
sudo ip addr add 10.10.10.12/24 dev enp0s8
sudo ip link set enp0s8 up
```

Verified:

```bash
ip -br addr
curl -i http://127.0.0.1:8000/health
```

Then tested from `web01`:

```bash
ping -c 4 10.10.10.12
nc -vz 10.10.10.12 8000
curl -i http://10.10.10.12:8000/health
```

The connection and HTTP health check succeeded after the network address was restored.

## Key Learning

A service can work locally while being unreachable remotely. Check the listening address and network interface configuration separately.
