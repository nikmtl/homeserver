# Implementation Plan

> [!Note]
> This document is the current plan for implementing the self-hosted homeserver. As i progress through the phases, I will update the document with the current status, any changes to the plan, and notes on the implementation process. This is a living document meant to guide the project from start to finish. This will be removed once the server is fully set up and running.

## Phase 1: Fresh OS + Hardening

**Objective**: Establish secure baseline before any service exposure

### Steps

1. **Install Ubuntu 24.04 LTS**
2. **SSH Hardening**
   - SSH key-only login (disable password auth)
   - Change SSH port to <ssh-port> (non-standard)
   - Disable root login
3. **Create Non-Root Sudo User**
4. **UFW Firewall**
   - Default deny incoming
   - Allow SSH (port <ssh-port>) only
5. **fail2ban**
   - Auto-ban brute-force attempts (3 tries in 10 min = 1 hour ban)
6. **Automatic Security Updates**
   - unattended-upgrades for critical patches


## Phase 2: Cloudflare Tunnel

**Objective**: Eliminate public 80/443 exposure for web services

### Key Points

- Install cloudflared
- Create tunnel via Cloudflare dashboard
- Routes all web traffic through Cloudflare (no inbound listening)


### Public Hostnames

| Service      | Domain            | Type                  |
| ------------ | ----------------- | --------------------- |
| Dokploy      | dokploy.example.com | HTTP → localhost:3000 |
| Trek         | trek.example.com    | HTTP → localhost:PORT |
| Recipe Cloud | recipes.example.com | HTTP → localhost:PORT |


## Phase 3: Dokploy

**Objective**: Deploy applications via unified Docker interface

### Setup

- One-liner install: `curl -sSL https://dokploy.com/install.sh | sh`
- Exposes dashboard on port 3000 (proxied through Cloudflare Tunnel)
- Deploy Trek, Recipe Cloud, Uptime Kuma via UI

## Phase 5: Restore & Verify

**Objective**: Restore user data and verify all services operational

### Tasks

1. Restore Trek data
2. Update DNS records (point to Cloudflare nameservers)
3. Update DDNS script (Minecraft mc.example.com only)
4. Verify all services accessible:
   - `https://trek.example.com`
   - `https://recipes.example.com`
   - `https://dokploy.example.com`
   - `https://hosting.example.com` (Pterodactyl)
   - `mc.example.com:25565` (direct port)

## Phase 6: Backups
**Objective**: Implement regular backups to local NAS

### Backup Plan
- Minecraft worlds: Weekly automated backups to NAS
- Trek data: Database snapshots before updates
- Retention: Keep backups for 90+ days
- Test restore procedure monthly to ensure backup integrity

---

## Optional: Pterodactyl for Minecraft

**Note**: Can skip if managing Minecraft directly; this is for managed hosting

### Setup

- Install Pterodactyl Panel + Wings (Docker)
- Panel runs behind Cloudflare Access (hosting.example.com)
- Wings port 8443 restricted to localhost/trusted IPs
- Minecraft game ports (25565-25570) open + port-forwarded

### Considerations

- Alternative: Direct port forward + SFTP without Pterodactyl
- Pterodactyl useful if managing multiple Minecraft instances

---

## Security Layers (Priority Order)

### 🔴 CRITICAL

1. **SSH Hardening + UFW**
   - Do first — protects all other infrastructure
   - Key-only auth, no root login, custom port, fail2ban, UFW, Automatic updates

2. **Cloudflare Tunnel**
   - Eliminates biggest attack surface (80/443)
   - Web services only accessible via Cloudflare
   - Attackers cannot reach server directly

3. **Cloudflare Access (Zero-Trust)**
   - Put in front of: Dokploy, Pterodactyl dashboards
   - Users authenticate via email/Google before login page visible
   - Even if service has vuln, attacker can't reach it

### 🟡 SHOULD

1. **Docker security basics**
   - Never expose container ports directly to 0.0.0.0 unless necessary — bind to 127.0.0.1 where possible 
   - Dokploy/Traefik handles routing, so most containers shouldn't have public ports at all
   - Keep Docker and images updated

2. **Secrets management**
   - No hardcoded secrets in docker-compose files
   - Use .env files, keep them out of Git
   - Your new GitHub docs repo — never commit API tokens, passwords, or keys

### 🟢 NICE

1. **Uptime Kuma** — Service monitoring + Discord/Telegram alerts
2. **Log Monitoring** — Weekly review of auth/fail2ban logs
3. **Dokploy has built-in resource monitoring**


---

## Minecraft-Specific Decisions

**Why not tunnel Minecraft?**
- Cloudflare Tunnel doesn't support UDP/game protocols
- Minecraft uses TCP but Cloudflare Tunnel is HTTP/HTTPS only
- Direct port forward is only option: TCP 25565-25570

This is why Minecraft is separate for now — it will be exposed directly to the internet, but with strong firewall rules and fail2ban monitoring.

---

## Completion Checklist

### Phase 1 ✓
- [x] Ubuntu 24.04 installed + updated
- [x] SSH key-only auth enabled
- [x] SSH port changed to <ssh-port>
- [x] UFW configured (allows SSH only)
- [x] fail2ban running
- [x] Non-root user created
- [x] Automatic updates configured

### Phase 2
- [ ] cloudflared installed + running
- [ ] Tunnel created + connected
- [ ] Public hostnames routed
- [ ] Cloudflare Access enabled on admin UIs
- [ ] Verified: no direct 80/443 access

### Phase 3
- [ ] Docker installed
- [ ] Dokploy running on port 3000
- [ ] Trek deployed
- [ ] Recipe Cloud deployed
- [ ] Uptime Kuma deployed
- [ ] Services accessible via HTTPS domains

### Phase 5
- [ ] Trek data restored
- [ ] DNS updated
- [ ] DDNS script running (Minecraft only)
- [ ] All services verified accessible
- [ ] Backups configured + tested

---

## References

- **Cloudflare Tunnel Docs**: https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/
- **Dokploy**: https://dokploy.io
  - Dokploy Cloudflare Tunnels Guide: https://docs.dokploy.com/docs/core/guides/cloudflare-tunnels
- **Ubuntu Security**: https://wiki.ubuntu.com/Security
- **Docker Security**: https://docs.docker.com/engine/security/
