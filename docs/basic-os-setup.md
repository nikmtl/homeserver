# Basic OS Setup
This document outlines the initial OS setup.

## OS 
`Ubuntu Server (minimized) 26.04 LTS` is chosen for its stability, security, and long-term support. The minimal installation reduces the attack surface and resource usage, providing a clean slate for the server setup.

### User Management
A non-root user with sudo privileges is created for daily administration tasks. This enhances security by preventing direct root access and allows for better control over permissions.

## SSH 

Connecting to the server is done via SSH on port <ssh-port> with key-based authentication. Root login and password authentication are disabled for security reasons.

A key pair is generated on the local machine using `ssh-keygen` and the public key is added to the server's `~/.ssh/authorized_keys` file for the non-root sudo user.

### Connecting
To connect to the server I use the following configuration in `~/.ssh/config`:

```
Host homeserver
    HostName [domain-to-ssh.com]
    User [username]
    IdentityFile ~/.ssh/id_ed25519
    ProxyCommand cloudflared access ssh --hostname %h
```

This allows for a simplified connection command, `ssh homeserver`, which uses the specified hostname, user, and key without needing to enter them each time. The `ProxyCommand` is used to route the SSH connection through Cloudflare Access for added security. For this to work, `cloudflared` must be installed on the local machine.

### Hardening SSH
To further harden SSH, the following settings are applied in `/etc/ssh/sshd_config`:
```
Port <ssh-port>                                   # Change SSH port to non-standard
PermitRootLogin no                          # Disable root login
PasswordAuthentication no                   # Disable password authentication (key-only login)
PubkeyAuthentication yes                    
AuthorizedKeysFile .ssh/authorized_keys     # Default location for authorized keys
X11Forwarding no                            # Disable X11 forwarding for security
```

## Laptop as Server
As I'm using my laptop as a server, there are some additional considerations:

### Lid Behavior
The laptop is configured to not suspend or hibernate when the lid is closed:
Change the following settings in `/etc/systemd/logind.conf`:
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

### Battery Management
To preserve battery health, the laptop is set to charge only up to 80% when plugged in. This can be configured using the `tlp` tool or through the BIOS/UEFI settings if supported by the hardware.
My settings in `/etc/tlp.conf`:
```
START_CHARGE_THRESH_BAT0=0
STOP_CHARGE_THRESH_BAT0=1
```
> To check battery status and thresholds, use `tlp-stat -b` to see current battery information and charging thresholds.

Another option would be to completely remove the battery and run the laptop on AC power only, but this may not be feasible for all laptop models and can have implications for power stability. 
At the moment of this documentation my battery capacity is at 86.5% (14.05.2026) and I will consider removing the battery if it drops significantly further.


## Further Basic Security Setup 
See [Fail2Ban](./security.md#fail2ban), [Automatic Security Updates](./security.md#automatic-security-updates), and [UFW Firewall](./security.md#ufw-firewall) for details on their setup.