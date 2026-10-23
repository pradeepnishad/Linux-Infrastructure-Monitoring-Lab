# Troubleshooting 08: Running FastAPI as a systemd Service

## Objective

Configure the FastAPI application on `app01` to run as a managed Linux service, start automatically at boot, and restart automatically if the application process fails.

## Environment

| Component | Configuration |
|---|---|
| Server | `app01` |
| Operating system | Ubuntu Server |
| Application directory | `/home/appadmin/infra-app` |
| Python virtual environment | `/home/appadmin/infra-app/venv` |
| Application entry point | `main:app` |
| Application server | Uvicorn |
| Listening address | `0.0.0.0:8000` |
| Environment file | `/home/appadmin/infra-app/.env` |
| Service name | `infra-app.service` |

## 1. Problem

FastAPI was running as a manually launched Uvicorn process. This meant application startup depended on manually executing the command in a terminal after a reboot or process termination.

The goal was to use `systemd` to manage the application lifecycle.

## 2. Initial Investigation

Identify the running Uvicorn process:

```bash
pgrep -af uvicorn
```

Inspect the application directory:

```bash
ls -la /home/appadmin/infra-app/
```

Verify that the application was listening on port 8000:

```bash
sudo ss -lntp | grep 8000
```

The investigation confirmed:

- The Python virtual environment existed.
- The `main.py` application file existed.
- The `.env` file existed.
- Uvicorn was listening on `0.0.0.0:8000`.

## 3. Stop the Manually Launched Process

The original Uvicorn process was running with PID `1390`.

It was stopped gracefully:

```bash
kill -TERM 1390
```

The port was then checked to ensure it was available before starting the systemd-managed instance:

```bash
sudo ss -lntp | grep 8000
```

If the command produces no output, no process is listening on port 8000.

## 4. Create the systemd Unit File

Created the service file:

```bash
sudo nano /etc/systemd/system/infra-app.service
```

Configuration:

```ini
[Unit]
Description=Infrastructure Monitor FastAPI
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=appadmin
WorkingDirectory=/home/appadmin/infra-app
EnvironmentFile=/home/appadmin/infra-app/.env
ExecStart=/home/appadmin/infra-app/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Configuration Explanation

- `Description`: identifies the service.
- `After=network-online.target`: orders service startup after the network-online target.
- `User=appadmin`: runs the application without root privileges.
- `WorkingDirectory`: sets the application's working directory.
- `EnvironmentFile`: loads application environment variables.
- `ExecStart`: launches Uvicorn using the virtual environment's executable.
- `Restart=on-failure`: restarts the application when it exits unsuccessfully.
- `RestartSec=5`: waits five seconds before restarting.
- `WantedBy=multi-user.target`: allows the service to start automatically during normal system boot.

## 5. Reload systemd and Enable the Service

Reloaded the systemd manager configuration:

```bash
sudo systemctl daemon-reload
```

Enabled and started the service:

```bash
sudo systemctl enable --now infra-app
```

The terminal confirmed that the service symlink was created under:

```text
/etc/systemd/system/multi-user.target.wants/infra-app.service
```

## 6. Verify Service Status

Executed:

```bash
sudo systemctl status infra-app --no-pager
```

### Observed Result

The screenshot confirmed:

- Service state: `active (running)`
- Service enabled for automatic startup: `enabled`
- Uvicorn process running under systemd
- Application startup completed successfully
- Application listening on `0.0.0.0:8000`

## 7. Test the Application Endpoints

### Health Endpoint

```bash
curl -i http://127.0.0.1:8000/health
```

Observed result:

```text
HTTP/1.1 200 OK
```

The response reported:

```json
{
  "status": "healthy"
}
```

### Server Inventory Endpoint

```bash
curl -i http://127.0.0.1:8000/servers
```

Observed result:

```text
HTTP/1.1 200 OK
```

The endpoint returned three inventory records:

- `web01` — `10.10.10.11`
- `app01` — `10.10.10.12`
- `db01` — `10.10.10.13`

This confirmed that the application could retrieve the inventory data successfully.

## 8. Troubleshooting Commands

Check service status:

```bash
sudo systemctl status infra-app --no-pager
```

Inspect recent logs:

```bash
sudo journalctl -u infra-app -n 50 --no-pager
```

Follow logs in real time:

```bash
sudo journalctl -u infra-app -f
```

Restart the service:

```bash
sudo systemctl restart infra-app
```

Stop the service:

```bash
sudo systemctl stop infra-app
```

Check whether automatic startup is enabled:

```bash
sudo systemctl is-enabled infra-app
```

## 9. Verification Status

| Test | Result |
|---|---|
| systemd unit created | Passed |
| Service enabled | Passed |
| Service active and running | Passed |
| `/health` returns HTTP 200 | Passed |
| `/servers` returns HTTP 200 | Passed |
| Inventory returns all three server records | Passed |
| Service starts successfully after reboot | Pending final reboot test |

## 10. Lessons Learned

1. `systemd` provides a consistent way to manage long-running Linux applications.
2. A Python virtual environment can be used directly from a systemd unit.
3. Running an application as a dedicated non-root user reduces unnecessary privileges.
4. A service can be enabled for automatic startup and separately checked for current running status.
5. Application health checks and application logs are essential troubleshooting tools.
6. HTTP 200 responses from both health and inventory endpoints provide evidence that the API is responding correctly.

## 11. Next Steps

- Reboot `app01` and verify that `infra-app.service` starts automatically.
- Test `/health` and `/servers` again after reboot.
- Test the endpoints through the Nginx reverse proxy on `web01`.
- Confirm the complete request path from the host browser to FastAPI and PostgreSQL.
- Add database backup and recovery procedures to the project documentation.
