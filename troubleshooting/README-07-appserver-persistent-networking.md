# Troubleshooting 07: Persistent Networking on app01

## Objective

Configure a persistent private IPv4 address on the application server so it can communicate with the web and database servers after a reboot.

## Lab Environment

| Setting | Value |
|---|---|
| Server | `app01` |
| Operating system | Ubuntu Server |
| NAT interface | `enp0s3` |
| Internal interface | `enp0s8` |
| NAT address | `10.0.2.15/24` |
| Private address | `10.10.10.12/24` |
| Application | FastAPI |
| Application port | `8000` |
| Database server | `db01` — `10.10.10.13` |

## 1. Problem

The application server's private IPv4 address was missing during troubleshooting. Consequently, other servers could not reliably reach FastAPI over the internal network.

The database server also temporarily lacked its private IPv4 address, and its PostgreSQL cluster was stopped.

## 2. Investigation

Inspect network interfaces:

```bash
ip -br addr
ip route
```

Inspect the FastAPI listener:

```bash
sudo ss -lntp | grep 8000
```

Inspect PostgreSQL connectivity from `app01`:

```bash
ping -c 4 10.10.10.13
nc -vz 10.10.10.13 5432
```

The investigation identified missing private IPv4 addresses on the internal interfaces. PostgreSQL was also found offline on `db01`.

## 3. Temporary Network Recovery

The private IP on `app01` was restored with:

```bash
sudo ip addr add 10.10.10.12/24 dev enp0s8
sudo ip link set enp0s8 up
```

On `db01`, the missing IP was restored with:

```bash
sudo ip addr add 10.10.10.13/24 dev enp0s8
sudo ip link set enp0s8 up
```

PostgreSQL was started using:

```bash
sudo pg_ctlcluster 18 main start
```

## 4. Permanent Netplan Configuration

The application server's configuration file was:

`/etc/netplan/00-installer-config.yaml`

The NAT interface was preserved, and the following configuration was added for the internal interface:

```yaml
enp0s8:
  dhcp4: false
  dhcp6: false
  addresses:
    - 10.10.10.12/24
```

The existing `enp0s3` DHCP configuration and its MAC address mapping were retained.

Validate the configuration with:

```bash
sudo netplan generate
sudo netplan try
```

## 5. Verification Results

The following checks were successful during troubleshooting:

- `app01` had `10.10.10.12/24` on `enp0s8`.
- `app01` could ping `db01` at `10.10.10.13`.
- TCP connectivity to PostgreSQL port `5432` succeeded.
- FastAPI's `/health` endpoint returned HTTP 200.
- FastAPI's `/servers` endpoint returned the three inventory records: `web01`, `app01`, and `db01`.

The configuration file explicitly contains the static address. A separate post-reboot verification is required to confirm persistence after the final configuration.

## 6. Lessons Learned

1. A service may be healthy locally but unreachable from another VM if the network interface lacks its private IPv4 address.
2. A successful ping does not prove that an application port is reachable; test TCP ports separately.
3. Temporary `ip addr add` commands do not persist across reboots.
4. Netplan must preserve the NAT adapter while configuring the internal interface.
5. PostgreSQL service status, network connectivity, and database authentication are separate checks.

## Next Steps

- Complete persistent networking on `db01`.
- Reboot-test all three VMs.
- Configure FastAPI to start automatically with systemd.
- Verify the complete request path: Nginx → FastAPI → PostgreSQL.
