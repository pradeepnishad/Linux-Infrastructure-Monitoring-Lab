# Troubleshooting 02: PostgreSQL Connection Refused

## Problem

The application VM returned:

```text
connection to server at "10.10.10.13", port 5432 failed:
Connection refused
```

## Environment

- Application VM: `app01`
- Application IP: `10.10.10.12`
- Database VM: `db01`
- Database IP: `10.10.10.13`
- Database: PostgreSQL 18
- Port: TCP 5432

## Investigation

### 1. Check PostgreSQL cluster status

On `db01`:

```bash
sudo systemctl status postgresql@18-main --no-pager
sudo pg_lsclusters
```

The cluster was online.

### 2. Check the listening socket

```bash
sudo ss -lntp | grep 5432
```

PostgreSQL was listening on:

```text
10.10.10.13:5432
```

### 3. Check firewall status

```bash
sudo ufw status verbose
sudo nft list ruleset
```

UFW was inactive, and no nftables rules were listed at the time of testing.

### 4. Test from the application VM

```bash
nc -vz 10.10.10.13 5432
```

The connection initially failed.

## Root Cause

The database service was running, but the application VM had lost its private IPv4 address. The missing address disrupted communication over the Internal Network.

## Resolution

Restored the application VM's private IPv4 address:

```bash
sudo ip addr add 10.10.10.12/24 dev enp0s8
sudo ip link set enp0s8 up
```

Then tested connectivity:

```bash
ping -c 4 10.10.10.13
nc -vz 10.10.10.13 5432
```

Both tests succeeded after network connectivity was restored.

## Result

The application VM successfully connected to PostgreSQL.

## Key Learning

A running database service does not guarantee remote connectivity. Always check the listener, client IP, routing, firewall, and PostgreSQL authentication rules.
