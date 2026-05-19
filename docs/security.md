# Security Documentation
This document outlines the security measures implemented for the server setup.

## UFW Firewall
The Uncomplicated Firewall (UFW) is configured to allow only SSH traffic on port <ssh-port> and deny all other incoming connections by default. This helps to protect the server from unauthorized access while still allowing secure remote management.

**UFW Configuration:**
| Type  | Port | Protocol | Note/Service |
| ----- | ---- | -------- | ------------ |
| Allow | <ssh-port> | tcp      | ssh          |

Default policy is set to deny all incoming connections and allow all outgoing connections.


> Command cheatsheet: \
> `sudo ufw status` to check firewall status \
> `sudo ufw allow/deny <port>` to allow/deny SSH \
> `sudo ufw default deny incoming` to set default policy \
> `sudo ufw enable` to activate the firewall

### Docker and UFW
By default, Docker bypasses UFW by directly modifying iptables rules. This means ports exposed by Docker containers are reachable from the public internet even when UFW is configured to deny all incoming traffic.

[UFW Docker](https://github.com/chaifeng/ufw-docker) patches UFW to properly 
control Docker traffic, blocking all container ports by default.

**Installation:**
```
sudo wget -O /usr/local/bin/ufw-docker \
  https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker
sudo chmod +x /usr/local/bin/ufw-docker
sudo ufw-docker install
sudo systemctl restart ufw
```

All Docker ports are blocked by default after installation. Since all inbound 
traffic is routed through Cloudflare Tunnel (outbound connection), no additional 
`ufw-docker allow` rules are needed.


## Fail2Ban
Fail2Ban is installed and configured to monitor SSH login attempts. 

The following configuration is applied to the SSH jail under `/etc/fail2ban/jail.local` to protect against brute-force attacks:
``` 
[sshd]
mode = aggressive
enabled = true
port = <ssh-port>
maxretry = 5
bantime = 1h
```

> Command cheatsheet: \
> `sudo systemctl enable fail2ban` to enable fail2ban on boot \
> `sudo systemctl start fail2ban` to start the fail2ban service \
> `sudo fail2ban-client status` to check overall status \
> `sudo fail2ban-client status sshd` to check SSH jail status \
> `sudo fail2ban-client set sshd unbanip <IP_ADDRESS>` to manually unban an IP address



## Automatic Security Updates
Automatic security updates are enabled to ensure the system is always up-to-date with the latest security patches.

```
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## SSH Hardening
See the [Basic OS Setup documentation](./basic-os-setup.md#ssh) for details on the SSH hardening configuration, including changing the default port, disabling root login, and enforcing key-based authentication.

## Exposing Services with Cloudflare Tunnel
To securely expose internal services to the internet without opening ports on the server, Cloudflare Tunnel is used. This creates an encrypted tunnel between the local server and Cloudflare's network. 

For more details on the Cloudflare Tunnel configuration, how the routing with the Tunnel and Reverse Proxy works, see the [Cloudflare setup documentation](/docs/cloudflare.md).

## Cloudflare Access
Cloudflare Access is enabled for all admin UIs (e.g., Dokploy dashboard) to add an additional layer of security. This requires users to authenticate through Cloudflare before accessing the admin interfaces, providing protection against unauthorized access even if the tunnel is exposed.

See the [Cloudflare setup documentation](/docs/cloudflare.md) for details on the Access configuration and policies.