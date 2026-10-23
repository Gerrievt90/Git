# Ubuntu Home Server Agent

A dependency-free, local health agent for an Ubuntu home server. It runs via a hardened systemd service every five minutes and writes one JSON health record per run. It is intentionally **local-only**: it neither opens a port nor sends server details to a third party.

## What it checks

- Persistent filesystem utilization (default warning threshold: 85%).
- One-minute load normalized by online CPU count (default: 1.50).
- systemd's overall system state.
- An optional administrator-defined health command, such as checking Jellyfin, Pi-hole, or a backup service.

A non-healthy run exits with status `1`, allowing systemd, monitoring tools, or future alert hooks to detect it.

## Install

Copy this repository to the server, then run:

```bash
sudo ./bin/setup-home-server-agent install
```

The installer creates an administrator-owned configuration at `/etc/home-server-agent/agent.conf`, installs the executable at `/usr/local/sbin/home-server-agent`, and enables a persistent systemd timer. It immediately performs the first check.

> Review the configuration before adding `CHECK_COMMAND`: it is deliberately executed as root, so only root should be able to edit `/etc/home-server-agent/agent.conf`.

For example, alert if Jellyfin is down:

```bash
sudoedit /etc/home-server-agent/agent.conf
# CHECK_COMMAND='systemctl is-active --quiet jellyfin.service'
sudo systemctl start home-server-agent.service
```

## Operate

```bash
sudo ./bin/setup-home-server-agent status
systemctl list-timers home-server-agent.timer
journalctl -u home-server-agent.service --since today
sudo tail -f /var/log/home-server-agent/health.jsonl
```

Each JSON line has a UTC timestamp, host, status, one-minute load, CPU count, and problem list. The last simple status is also kept in `/var/lib/home-server-agent/status`.

## Remove

```bash
sudo ./bin/setup-home-server-agent uninstall
```

Uninstall stops the timer and removes runtime files, units, executable, state, and logs. It preserves the configuration as `/etc/home-server-agent/agent.conf.bak` for recovery.
