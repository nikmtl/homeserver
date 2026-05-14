# Security Documentation
This document outlines the security measures implemented for the server setup.

## UFW Firewall
The Uncomplicated Firewall (UFW) is configured to allow only SSH traffic on port <ssh-port> and deny all other incoming connections by default. This helps to protect the server from unauthorized access while still allowing secure remote management.

**UFW Configuration:**
| Type  | Port | Protocol | Note/Service                                  |
| ----- | ---- | -------- | --------------------------------------------- |
| Allow | <ssh-port> | tcp      | ssh                                           |
| Allow | 80   | tcp      | HTTP (Temporary for Cloudflare Tunnel setup)  |
| Allow | 443  | tcp      | HTTPS (Temporary for Cloudflare Tunnel setup) |
| Allow | 3000 | tcp      | Dokploy (Temporary for Cloudflare Tunnel setup) |

Default policy is set to deny all incoming connections and allow all outgoing connections.


> Command cheatsheet: \
> `sudo ufw status` to check firewall status \
> `sudo ufw allow/deny <port>` to allow/deny SSH \
> `sudo ufw default deny incoming` to set default policy \
> `sudo ufw enable` to activate the firewall


## Fail2Ban
Fail2Ban is installed and configured to monitor SSH login attempts. 

The following configuration is applied to the SSH jail under `/etc/fail2ban/jail.local` to protect against brute-force attacks:
``` 
[sshd]
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