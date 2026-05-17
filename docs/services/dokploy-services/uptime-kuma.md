# Service –  Uptime Kuma
**Uptime Kuma** is a monitoring tool that allows to keep track of the uptime and performance of the services running on the server.

## Setup (Through Dokploy)
I am using the template provided by Dokploy for Uptime Kuma: 

**General:** \
Service Type: `Application` \
Application Name: `uptime-kuma` \
Dokploy Path: `Infrastructure/uptime-kuma` \
Provider: `Raw (Docker Compose)` \ 
Domain: `status.example.com`

**Docker Compose:** \
```yaml
version: "3.8"
services:
  uptime-kuma:
    image: louislam/uptime-kuma:2.1.0
    restart: always
    volumes:
      - uptime-kuma-data:/app/data
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  uptime-kuma-data:
```

## Configuration (In Uptime Kuma)