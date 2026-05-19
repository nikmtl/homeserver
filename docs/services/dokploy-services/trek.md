# Service – Trek 
**Trek** is a travel planner application that allows to organize and plan trips, including itinerary management, packing lists, and travel documentation.

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