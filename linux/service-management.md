# Linux Service Management

Context: [documentation/04](../documentation/04-ec2-and-nginx-deployment.md).

> **Evidence status:** the only service evidence is the default Nginx welcome page ([`nginx-default-page.png`](../screenshots/ec2-linux/nginx-default-page.png)). No `systemctl` output was captured. The commands below are standard references, **not** a record of what was run.

## Concepts

Ubuntu 24.04 uses `systemd`. A *unit* (for example `nginx.service`) describes how to start, stop and supervise a process. `enable` makes a unit start at boot; `start` runs it now. Logs are in the journal.

## Common commands (illustrative)

```bash
sudo systemctl status nginx        # current state and recent log lines
sudo systemctl enable --now nginx  # start now and at every boot
sudo systemctl restart nginx       # stop and start (brief interruption)
sudo systemctl reload nginx        # re-read configuration without dropping connections
sudo nginx -t                      # validate configuration before reloading
sudo journalctl -u nginx --since "1 hour ago"
sudo ss -tlnp | grep -E ':80|:443' # confirm listening ports
```

## Operational notes

- Always run `nginx -t` before `reload` or `restart`.
- Nginx logs: `/var/log/nginx/access.log` and `/var/log/nginx/error.log`.
- A service that is `active` but unreachable from outside points to the network path (route, security group, NACL), not the service. See [troubleshooting](../documentation/12-troubleshooting-and-lessons-learned.md).
- Apply package updates with `sudo apt update && sudo apt upgrade`, then restart services or reboot if indicated.
