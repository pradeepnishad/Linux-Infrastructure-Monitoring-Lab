# Linux Infrastructure Monitoring Lab

A hands-on Linux infrastructure project built using VirtualBox, Prometheus, Grafana, Node Exporter, and PostgreSQL. The project focuses on Linux server monitoring, metrics visualization, alert evaluation, networking, SSH troubleshooting, and database backup and recovery validation.

## Project Overview

In this project, I configured and tested a Linux monitoring environment using virtual machines. I collected Linux host metrics with Node Exporter, used Prometheus to scrape and query metrics, created a Grafana monitoring dashboard, tested Prometheus alert rules, and validated PostgreSQL backup restoration.

I also documented the networking, service configuration, and connectivity issues encountered during implementation.

## Technologies Used

- **Linux:** Server administration and troubleshooting
- **Oracle VirtualBox:** Virtual machine environment
- **SSH:** Remote Linux access
- **Node Exporter:** Linux host metrics
- **Prometheus:** Metrics collection, querying, and alert evaluation
- **Grafana:** Monitoring dashboards and visualization
- **PostgreSQL:** Database backup and recovery testing
- **systemd:** Linux service management
- **Markdown:** Technical documentation

## Quick-Start Setup

Follow these steps to access and verify the monitoring environment.

### Prerequisites

- Oracle VirtualBox installed on the host machine.
- The configured Linux virtual machines available.
- Prometheus, Grafana, and Node Exporter installed on their configured machines.
- The virtual machines' network configuration and port forwarding set up as documented in the project.

### 1. Start the Virtual Machines

Start the configured Linux virtual machines in VirtualBox and confirm that they have booted successfully.

### 2. Verify the Linux Network

On the relevant Linux VM, run:

```bash
ip addr
ip route
hostname -I
```

Check the assigned IP addresses and routes. Use the private-network IP addresses configured for your monitoring targets.

### 3. Check the Monitoring Services

On the VM hosting the relevant services, verify their status:

```bash
sudo systemctl status prometheus
sudo systemctl status grafana-server
sudo systemctl status prometheus-node-exporter
```

Run only the commands for services installed on that particular VM.

### 4. Open the Monitoring Interfaces

If the existing VirtualBox port-forwarding rules are enabled, open these addresses on the host machine:

| Service | URL |
|---|---|
| Prometheus | http://127.0.0.1:9090 |
| Grafana | http://127.0.0.1:3000 |

These addresses use the host ports configured in the lab. If you change the port-forwarding rules, use the corresponding host ports instead.

### 5. Verify Prometheus Targets

Open the Prometheus Targets page:

http://127.0.0.1:9090/targets

Confirm that the configured Linux monitoring targets are present and check their status.

### 6. Open the Grafana Dashboard

Open Grafana and navigate to the **Linux Infrastructure Monitoring** dashboard.

Check the panels for server availability, CPU utilization, memory utilization, disk utilization, and network traffic.

**Note:** If a target is down or a panel has no data, verify the VM is running, Node Exporter is active, the target IP and port are correct, and the private network allows connectivity.

## Monitoring Architecture

```text
Linux Virtual Machines
         |
         v
   Node Exporter
         |
         v
    Prometheus
         |
         v
      Grafana
         |
         v
Linux Infrastructure
Monitoring Dashboard
```

## Work Completed

### 1. Linux Networking and SSH

- Inspected Linux network interfaces and IP addresses.
- Worked with NAT and private networking in VirtualBox.
- Configured and tested SSH access to Linux virtual machines.
- Used host port forwarding to access services.
- Troubleshot IP addressing and network connectivity issues.
- Documented the networking and SSH troubleshooting process.

### 2. Node Exporter

- Installed and configured Prometheus Node Exporter.
- Managed the Node Exporter service using `systemctl`.
- Enabled the service to start automatically.
- Verified that the service was running and exposing host metrics.

### 3. Prometheus

- Configured Prometheus scrape targets for Linux host metrics.
- Used Prometheus to collect and query metrics.
- Verified target availability through the Prometheus interface.
- Checked Prometheus service readiness.
- Configured and evaluated alert rules for server availability and resource usage.

### 4. Grafana Dashboard

Created a dashboard named **Linux Infrastructure Monitoring** with panels for:

- Server availability
- CPU utilization
- Memory utilization
- Disk utilization
- Network receive traffic
- Network transmit traffic

Configured Prometheus as the Grafana data source and verified that the dashboard could query monitoring data.

### 5. Prometheus Alert Evaluation

Configured and tested the following alert rules:

- `InstanceDown`
- `HighCPUUsage`
- `HighMemoryUsage`
- `DiskSpaceLow`

Observed alert states during testing, including an instance-down condition. Verified that the affected target returned to an up state after troubleshooting.

### 6. PostgreSQL Backup and Recovery

- Created a PostgreSQL custom-format database backup.
- Calculated and compared SHA-256 checksums for backup copies.
- Restored the backup into a separate test database.
- Queried the restored `server_inventory` table.
- Verified that the restored table contained three rows.

## Troubleshooting Documentation

The `troubleshooting/` directory contains documentation for the issues encountered during the project:

- Nginx host access
- PostgreSQL connection problems
- PostgreSQL configuration path
- Missing application-server IP
- Nginx reverse proxy
- Temporary IP addresses
- Persistent application-server networking
- FastAPI systemd service

## Project Documentation

1. Linux Infrastructure Monitoring Lab
2. Infrastructure Architecture
3. Linux Networking and SSH Troubleshooting
4. Node Exporter - Linux Host Metrics
5. Prometheus Metrics Collection and Alert Evaluation
6. Grafana Linux Infrastructure Monitoring Dashboard
7. Prometheus Alert Rules and Failure Detection
8. PostgreSQL Backup and Recovery Validation

## Key Learnings

- Linux server administration and service management
- SSH connectivity and network troubleshooting
- Host metrics collection with Node Exporter
- Prometheus configuration and PromQL queries
- Grafana dashboard creation and visualization
- Prometheus alert rule evaluation
- PostgreSQL backup, checksum verification, and restoration testing
- Technical documentation and troubleshooting

## Project Summary

