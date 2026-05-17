# Dokploy 
**Dokploy** is a self-hosted deployment dashboard that allows to manage and deploy Docker containers with ease. It provides a user-friendly interface to monitor and control applications, making it easier to keep track of services and their status. This is the main tool for managing all services on the server, including web applications, databases, and other containerized services.

## Setup 
Dokploy is installed using the official installation script provided in their documentation. The installation process involves running a single command that sets up Dokploy and its dependencies.

```bash
curl -fsSL https://dokploy.com/install.sh | sh
```
After installation, Dokploy runs on `port 3000` and an admin account should be created to access the dashboard. 

> For me the timezone was not set correctly by default, so I had to set it manually:
> ```bash
> docker service update --env-add TZ=Europe/Berlin dokploy
> ```

## Custom Settings
After setting up the Cloudflare Tunnel connection, the only setting I had to change was the domain for the dashboard. This is done in the Dokploy dashboard under `Settings -> Web Server`.

As I'm using Cloudflare Tunnel, the protocol is set to HTTP since Cloudflare will handle SSL/TLS termination at the edge.

## Essential Services 
There are some essential services that are set up in Dokploy before exposing any applications:

| Service                                       | Description              | Status |
| --------------------------------------------- | ------------------------ | ------ |
| [Cloudflared](./dokploy-services/cloudflared) | Cloudflare Tunnel client | 🟢      |

## Custom Services
Custom services are added to Dokploy as needed, depending on the applications and services wanted to run on the server. Each service is configured with its own settings, such as environment variables, ports, and volumes, to ensure it runs correctly within the Docker environment managed by Dokploy.