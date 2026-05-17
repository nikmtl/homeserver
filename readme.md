![banner](assets/readme_banner.png)
> [!WARNING]
> At the moment the server is not fully set up and running. See the [plan](./docs/plan.md) for details on the implementation steps and current status.

This is the documentation for a self-hosted homeserver running own projects, open source web services and applications with security-first architecture using Cloudflare Tunnel and Dokploy.

A Minecraft server may also be hosted on the same machine in the future.

> [!NOTE]
> This is a personal project for self-hosting web applications. The documentation is primarily for my own reference but may be useful to others interested in similar setups. If you have experience with self-hosting, Cloudflare Tunnel, or Dokploy, please feel free to share your insights by [creating an issue](https://github.com/nikmtl/homeserver/issues/new/choose).
> If you have a security concern, see [security](./security.md).

## Services
| Service                               | Description                       | Status | Location                           | Port |
| ------------------------------------- | --------------------------------- | ------ | ---------------------------------- | ---- |
| [Dokploy](./docs/services/dokploy.md) | Docker-based deployment dashboard | 🟢      | dash.example.com (Behind Cloudflare Access) | 3000 |

All other services are managed by Dokploy and are listed under the Dokploy section.

## Further Documentation

| Document       | Description                        | Link                                               |
| -------------- | ---------------------------------- | -------------------------------------------------- |
| Basic OS Setup | Basic setup like OS and SSH        | [docs/basic-os-setup.md](./docs/basic-os-setup.md) |
| Security       | Security measures                  | [docs/security.md](./docs/security.md)             |
| Cloudflare     | All settings related to Cloudflare | [docs/cloudflare.md](./docs/cloudflare.md)         |