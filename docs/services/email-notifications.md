# Email Notifications (Maileroo)

All outbound email from the server is sent through **[Maileroo](https://maileroo.com/)**, a transactional email / SMTP relay service. Using a dedicated relay gives proper deliverability a custom mail server would just not provide.

The sending domain is verified in Maileroo with the SPF, DKIM, and DMARC DNS records it provides, added in Cloudflare (see [cloudflare.md](../cloudflare.md)).

## SMTP Settings

The same SMTP credentials are reused by every service:

```
Host:       smtp.maileroo.com
Port:       587
Encryption: STARTTLS
Username:   <maileroo-smtp-username>
Password:   <maileroo-smtp-password>
From:       no-reply@<sending-domain>
```

Credentials are created in the Maileroo dashboard under **SMTP** and stored per service as environment variables / config 

## Services Using It

| Service                                          | Purpose of email                                    |
| ------------------------------------------------ | --------------------------------------------------- |
| [Dokploy](./dokploy.md)                          | Deployment / build failure notifications            |
| [Trek](./dokploy-services/trek.md)               | Account verification and notification mail          |
| [Nextcloud](./dokploy-services/nextcloud.md)     | Share notifications, password resets, activity mail |

### Dokploy

`Settings -> Notifications -> Email`. Fill in the SMTP settings above.

### Trek
`Admin -> Notifications -> Email`. Fill in the SMTP settings above.

### Nextcloud
`Administration settings -> Basic settings -> Email server`. Fill in the SMTP settings above.
