# Implementation Plan

> [!Note]
> This document is the current plan for implementing the self-hosted homeserver. As I progress through the phases, I will update the document with the current status, any changes to the plan, and notes on the implementation process. This is a living document meant to guide the project from start to finish. This will be removed once the server is fully set up and running.

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


## Phase 2: Dokploy

**Objective**: Install Dokploy before exposing any applications

### Key Points

- Install Dokploy and confirm the dashboard is running locally
- Keep internal application traffic on HTTP and let Cloudflare handle TLS termination
- Prepare Dokploy's domain and routing configuration before exposing services


### Initial Setup

- One-liner install: `curl -sSL https://dokploy.com/install.sh | sh`
- Expose the dashboard on port 3000 for local access
- Configure Dokploy domain settings so later tunnel routes can point to Traefik correctly


## Phase 3: Cloudflare Tunnel

**Objective**: Create the outbound-only tunnel after Dokploy is available

### Setup

- Install cloudflared as a Dokploy-managed service
- Create the tunnel via the Cloudflare dashboard
- Route traffic through Cloudflare without opening inbound 80/443 ports
- Use HTTP for internal connections; Cloudflare handles SSL/TLS termination at the edge


### Public Hostnames

| Service      | Domain            | Type                      |
| ------------ | ----------------- | ------------------------- |
| Dokploy      | dokploy.example.com | HTTP → dokploy-traefik:80 |
| Trek         | trek.example.com    | HTTP → localhost:PORT     |
| Recipe Cloud | recipes.example.com | HTTP → localhost:PORT     |


## Phase 4: Services

**Objective**: Deploy applications through Dokploy once the tunnel is in place

### Setup

- Deploy Trek, Recipe Cloud, and Uptime Kuma via the Dokploy UI
- Configure each service to use the domain it should expose through Cloudflare Tunnel
- Add Pterodactyl only if Minecraft management is needed

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
- Cloudflare Tunnel doesn't support UDP; Bedrock edition uses UDP and cannot be proxied through a standard HTTP tunnel.
- Java edition uses TCP (default port 25565), but exposing game ports through Cloudflare typically requires Cloudflare Spectrum or an Enterprise feature; for most setups direct port forwarding is the practical option.
- Therefore Minecraft is handled separately and will be exposed directly to the internet, protected by strict firewall rules and monitoring (fail2ban, port restrictions).

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
- [x] Dokploy installed and running
- [x] Dokploy dashboard accessible locally

### Phase 3
- [x] cloudflared installed + running
- [x] Tunnel created + connected
- [x] Public hostnames routed
- [x] Cloudflare Access enabled on admin UIs
- [x] Verified: no direct 80/443 access

### Phase 4
- [x] Dokploy running on port 3000
- [ ] Trek deployed
- [ ] Recipe Cloud deployed
- [ ] Uptime Kuma deployed
- [x] Services accessible via HTTPS domains

### Phase 5
- [ ] Trek data restored
- [x] DNS updated
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
