# Cloudflare 
This document outlines all the configuration of Cloudflare and how Cloudflare Tunnel integrates with the server setup. It covers the domain setup, DNS records, tunnel configuration, and how the routing works with the Traefik reverse proxy to expose internal services to the internet.

## Domain Setup
The domain `example.com` is managed by Cloudflare and is used for all public-facing services. 

### DNS Record
_Cloudflare Dashboard → your domain → DNS → Records_

**Wildcard Subdomain Record** \
Type: `CNAME` / `Tunnel` \
Name: `*` \
Content: `<tunnel-id>.cfargotunnel.com` \
TTL: `Auto` \
Proxy status: `Proxied` 

This wildcard record ensures that all subdomains of `example.com` are routed through the Cloudflare Tunnel, allowing for flexible service exposure without needing to create individual DNS records for each service. See [Routing with Cloudflare Tunnel and Traefik Reverse Proxy](#routing-with-cloudflare-tunnel-and-traefik-reverse-proxy) for how this integrates with the Traefik reverse proxy to route traffic to the appropriate internal services based on the subdomain.

**Specific SSH Subdomain Record** \
Type: `CNAME` / `Tunnel` \
Name: `ssh` \
Content: `<tunnel-id>.cfargotunnel.com` \
TTL: `Auto` \
Proxy status: `Proxied` 

SSH has its own subdomain to bypass the Traefik reverse proxy and route directly to the server.

> To get the tunnel ID: _Zero Trust → Networks → Connectors → click your tunnel → copy the ID from the URL or the tunnel details_

### SSL/TLS Settings
_Cloudflare Dashboard → your domain → SSL/TLS → Overview_

**SSL/TLS Encryption Mode**: `Full` \
This ensures that Cloudflare connects to the server using a secure connection, and the server must have a valid SSL certificate.

## Cloudflare Tunnel
_Zero Trust -> Network -> Connections_

Cloudflare Tunnel is set up to securely expose internal services without opening inbound ports on the server.

### Tunnel Configuration

#### Overview:

- **Tunnel Name**: `homeserver` 
- **Tunnel Type**: `cloudflared`

After [cloudflared](/docs/services/dokploy-services/cloudflared) is installed on the server and the Tunnel ID is set as an environment variable, the server shows as a connector in the Cloudflare dashboard. The tunnel is configured to start on boot and automatically reconnect if the connection drops.

#### Publish application routes:

- **SSH**: `ssh.example.com` → `<Local-Server-IP>:<ssh-port>` (bypasses Traefik for SSH access)
- **Traefik**: `*.example.com` →  `dokploy-traefik:80` (routes all other subdomains to Traefik)

Traffic flow: Cloudflare → Tunnel → Traefik → Services on Dokploy

### Routing with Cloudflare Tunnel and Traefik Reverse Proxy

All traffic through the wildcard domain `*.example.com` is routed to Traefik, which forwards requests to individual services based on the domain configured in Dokploy.

**Routing Flow:**
1. Request arrives at `app.example.com`
2. Cloudflare matches wildcard DNS record and routes through tunnel to `dokploy-traefik:80`
3. Traefik reads the `Host` header and matches it against domains configured in Dokploy
4. Request is forwarded to the appropriate service container

**Two Routing Options:**
- **Via Traefik (all subdomains except SSH):** Domain-based routing, multiple apps on one tunnel, Dokploy domain features apply
- **Direct Container Access (SSH):** Bypasses Traefik, direct tunnel to `<Local-Server-IP>:<ssh-port>`

## Cloudflare Access
Cloudflare Access is a service that provides an additional layer of security by requiring users to authenticate through Cloudflare before accessing protected resources.

### Access Configuration:
| Name    | Destination    | Access Policy | Type        | Session Duration |
| ------- | -------------- | ------------- | ----------- | ----------------- |
| Dokploy | dash.example.com | Only me       | Self-hosted | 24h               |
| SSH     | ssh.example.com  | Only me       | Self-hosted | 24h               |