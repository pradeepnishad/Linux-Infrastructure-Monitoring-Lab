# Troubleshooting 01: Accessing Nginx from the Host PC

## Problem

Nginx was installed and running on the `web01` Ubuntu VM, but opening `http://10.10.10.11` in the host computer's browser did not work.

## Environment

- Virtualization: Oracle VirtualBox
- Web server: Ubuntu Server
- Web server software: Nginx
- Internal IP: `10.10.10.11`
- Guest HTTP port: `80`

## Root Cause

The VM used VirtualBox NAT and an Internal Network. The host computer could not directly access the guest's Internal Network IP.

## Solution

Configured VirtualBox NAT port forwarding.

| Setting | Value |
|---|---|
| Protocol | TCP |
| Host IP | `127.0.0.1` |
| Host port | `8080` |
| Guest IP | Blank |
| Guest port | `80` |

## Verification

Check the Nginx service:

```bash
sudo systemctl status nginx
```

Test HTTP locally:

```bash
curl -I http://127.0.0.1
```

Open this URL on the host computer:

```text
http://127.0.0.1:8080
```

## Result

The default Nginx welcome page became accessible from the host computer.

## Key Learning

VirtualBox NAT does not automatically expose guest services to the host. Port forwarding provides a controlled way to access a service running inside a VM.
