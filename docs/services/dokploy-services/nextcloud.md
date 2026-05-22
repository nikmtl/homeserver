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
NEXTCLOUD_DOMAIN=cloud.example.com
REDIS_HOST=nextcloud_redis
MYSQL_DATABASE=nextcloud
MYSQL_USER=nextcloud
```

## Settings and Configuration
After setting up the service, there are a few additional configurations to be made:


In the Nextcloud configuration file (`config/config.php`), add the following lines to set the trusted proxies:
```php
  'overwriteprotocol' => 'https',
  'overwritehost' => 'cloud.example.com',
  'trusted_proxies' => ['172.16.0.0/12', '10.0.0.0/8'],
  'forwarded_for_headers' => ['HTTP_X_FORWARDED_FOR'],
  'maintenance_window_start' => '1',
  'default_phone_region' => 'DE',
```
For that u can use the following commands:

```bash
docker exec -u www-data <nextcloud_container> php occ config:system:set overwriteprotocol --value="https"
docker exec -u www-data <nextcloud_container> php occ config:system:set overwritehost --value="cloud.example.com"
docker exec -u www-data <nextcloud_container> php occ config:system:set trusted_proxies 0 --value="172.16.0.0/12"
docker exec -u www-data <nextcloud_container> php occ config:system:set trusted_proxies 1 --value="10.0.0.0/8"
docker exec -u www-data <nextcloud_container> php occ config:system:set forwarded_for_headers 0 --value="HTTP_X_FORWARDED_FOR"
docker exec -u www-data <nextcloud_container> php occ config:system:set maintenance_window_start --type=integer --value=1
docker exec -u www-data <nextcloud_container> php occ config:system:set default_phone_region --value="DE"

**Mimetype Migration**
To ensure that Nextcloud correctly identifies file types, you may need to perform a mimetype migration. This can be done by running the following command inside the Nextcloud container:
```bash
docker exec -u www-data <nextcloud_container> php occ maintenance:repair --include-expensive
```

**Transactional File Locking (Redis)**
To enable transactional file locking using Redis, run these commands inside the Nextcloud container:
```bash
docker exec -u www-data <nextcloud_container> php occ config:system:set memcache.locking --value="\OC\Memcache\Redis"
docker exec -u www-data <nextcloud_container> php occ config:system:set redis host --value="nextcloud_redis"
docker exec -u www-data <nextcloud_container> php occ config:system:set redis port --type=integer --value=6379
```