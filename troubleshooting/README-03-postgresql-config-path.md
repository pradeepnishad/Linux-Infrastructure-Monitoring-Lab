# Troubleshooting 03: Finding the Correct PostgreSQL Configuration

## Problem

Opening this file displayed an empty Nano editor:

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

## Root Cause

The installed PostgreSQL version was 18, not 16. Opening a nonexistent path in Nano can create a new, empty buffer instead of opening the active configuration.

## Investigation

List installed PostgreSQL configuration directories:

```bash
ls /etc/postgresql/
```

Find the active configuration file through PostgreSQL itself:

```bash
sudo -u postgres psql -tAc "SHOW config_file;"
```

Expected path in this environment:

```text
/etc/postgresql/18/main/postgresql.conf
```

Find the authentication file:

```bash
sudo -u postgres psql -tAc "SHOW hba_file;"
```

## Resolution

Opened the actual configuration file:

```bash
sudo nano /etc/postgresql/18/main/postgresql.conf
```

The listening address was configured as:

```text
listen_addresses = '10.10.10.13'
```

The client authentication file contained:

```text
host    infra_monitor    infra_app    10.10.10.12/32    scram-sha-256
```

Reloaded PostgreSQL after changing authentication rules:

```bash
sudo systemctl reload postgresql
```

Restarted PostgreSQL when changing `listen_addresses`, because that setting requires a restart:

```bash
sudo systemctl restart postgresql
```

## Verification

```bash
sudo pg_lsclusters
sudo ss -lntp | grep 5432
```

## Key Learning

Never assume a configuration path from a tutorial. Ask the running service for its actual configuration path and verify the installed version first.
