# Service – Trek 
**Trek** is a travel planner application that allows to organize and plan trips, including itinerary management, packing lists, and travel documentation.

See the [Trek GitHub repository](https://github.com/mauriceboe/TREK) for more information about the application, its features, and configuration options.

## Setup (Through Dokploy)
Trek is deployed as a custom service in Dokploy.

**General:** \
Service Type: `Compose/DockerCompose` \
Application Name: `trek` \
Dokploy Path: `Trek/trek` \
Provider: `Raw (Docker Compose)` \
Domain: `trek.example.com`

**Docker Compose:** 
```yaml
services:
  app:
    image: mauriceboe/trek:latest
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETUID
      - SETGID
    tmpfs:
      - /tmp:noexec,nosuid,size=64m
    environment:
      - NODE_ENV=production
      - PORT=3000
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}
      - TZ=${TZ:-UTC}
      - LOG_LEVEL=${LOG_LEVEL:-info}
      - ALLOWED_ORIGINS=${ALLOWED_ORIGINS}
      - APP_URL=${APP_URL}
      - TRUST_PROXY=1
      - COOKIE_SECURE=false
    volumes:
      - trek-data:/app/data
      - trek-uploads:/app/uploads
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 15s

volumes:
  trek-data:
  trek-uploads:
  ```

**Environment Variables:**
```
ENCRYPTION_KEY=        ← generate with: openssl rand -hex 32
APP_URL=https://trek.example.com
ALLOWED_ORIGINS=https://trek.example.com
TZ=Europe/Berlin
```

> [!NOTE]
> `COOKIE_SECURE=false` is required because Cloudflare Tunnel terminates TLS at the edge — Traefik and Trek only ever see plain HTTP internally. Without it, the session cookie's `Secure` flag causes login to fail with "Access token required". Safe here since the full path (browser↔Cloudflare via TLS, Cloudflare↔server via the tunnel's own encryption) is already encrypted.