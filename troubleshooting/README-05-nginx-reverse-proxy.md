# Troubleshooting 05: Nginx Reverse Proxy to FastAPI

## Problem

After configuring Nginx to forward requests to FastAPI, these URLs stopped working:

```text
http://127.0.0.1:8080/health
http://127.0.0.1:8080/servers
```

Nginx was running, but a direct upstream request timed out.

## Intended Architecture

```text
Host browser
    |
    v
web01 — Nginx :80
    |
    v
app01 — FastAPI :8000
    |
    v
db01 — PostgreSQL :5432
```

## Investigation

On `web01`:

```bash
sudo nginx -t
sudo systemctl status nginx --no-pager
curl -i --connect-timeout 5 http://10.10.10.12:8000/health
```

The upstream request timed out.

On `app01`:

```bash
ip -br addr
sudo ss -lntp | grep 8000
curl -i http://127.0.0.1:8000/health
```

FastAPI was running and returned HTTP 200 locally, but the private IPv4 address was missing.

## Root Cause

The reverse proxy configuration was not the primary problem. The web VM could not reliably reach the upstream because the internal IPv4 addresses were missing or misconfigured.

## Resolution

Restored the private IPv4 address on `app01`:

```bash
sudo ip addr add 10.10.10.12/24 dev enp0s8
sudo ip link set enp0s8 up
```

Restored the web VM's private address when it was found missing:

```bash
sudo ip addr add 10.10.10.11/24 dev enp0s8
sudo ip link set enp0s8 up
```

On `web01`, tested upstream connectivity:

```bash
ping -c 4 10.10.10.12
nc -vz 10.10.10.12 8000
curl -i http://10.10.10.12:8000/health
```

The tests succeeded.

## Reverse Proxy Configuration

The Nginx server block's `location /` was configured to forward requests:

```nginx
location / {
    proxy_pass http://10.10.10.12:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Validate and reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Verification

From the host PC:

```text
http://127.0.0.1:8080/health
http://127.0.0.1:8080/servers
```

The upstream health endpoint was confirmed to return HTTP 200 from `web01`. The final browser requests through Nginx should also be verified.

## Key Learning

When a reverse proxy returns an error, test the upstream directly from the proxy server before changing the proxy configuration.
