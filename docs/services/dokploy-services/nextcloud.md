# Service – Nextcloud
**Nextcloud** is a self-hosted cloud storage solution that allows to store and access files, calendars, contacts, and more from any device. It provides a secure and private alternative to commercial  cloud services, giving full control over data and privacy.

## Setup (Through Dokploy)
I am using the template provided by Dokploy for Nextcloud:

**General:** \
Service Type: `Compose/DockerCompose` \
Application Name: `nextcloud` \
Dokploy Path: `Nextcloud/nextcloud` \
Provider: `Raw (Docker Compose)` \
Domain: `cloud.example.com`

**Docker Compose:** 
```yaml
services:
  nextcloud:
    image: nextcloud:stable
    restart: always
    volumes:
      - nextcloud_data:/var/www/html
      - ../files/fix-nextcloud.sh:/usr/local/bin/fix-nextcloud.sh:ro
    environment:
      - MYSQL_HOST=nextcloud_db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    depends_on:
      - nextcloud_db
      - nextcloud_redis

  nextcloud_db:
    image: mariadb:10.11
    restart: always
    volumes:
      - nextcloud_db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}

  nextcloud_redis:
    image: redis:alpine
    restart: always

volumes:
  nextcloud_data:
  nextcloud_db_data:
  ```

**Environment Variables:**
```
MYSQL_ROOT_PASSWORD=        ← generate with: openssl rand -hex 32
MYSQL_PASSWORD=             ← generate with: openssl rand -hex 32
MYSQL_DATABASE=nextcloud
MYSQL_USER=nextcloud
DEFAULT_PHONE_REGION=DE
NEXTCLOUD_DOMAIN=cloud.example.com
OVERWRITEPROTOCOL=https
TRUSTED_PROXIES=10.0.0.0/8 172.16.0.0/12
REDIS_HOST=nextcloud_redis
MYSQL_DATABASE=nextcloud
MYSQL_USER=nextcloud
```