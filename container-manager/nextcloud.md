# Nextcloud

- Create folder `/volume1/docker/nextcloud` and create or update here the following:
  - An `.env` file with your data (e.g.):
    ```text
    HOST_NAME=your.domain
    HOST_IP=192.168.1.1
    WEB_PORT=8080
    ```
  - Passwords for DB (random/strong) & SMTP server
    - `db_password.txt`
    - `smtp_password.txt`
  - The official suggested [nginx.conf](https://github.com/nextcloud/docker/blob/master/.examples/docker-compose/with-nginx-proxy/mariadb/fpm/web/nginx.conf) file 
- **Container Manager**
- Create new project **nextcloud** on above folder with the content below
- Control Panel -> Login Portal -> Advanced -> **Reverse Proxy**
- Create a new proxy for Nextcloud, e.g. https://your.domain -> http://192.168.1.1:8080
- Enjoy **Nextcloud**

## Assumptions

- You have installed **Container Manager**
- The folder `/volume1/docker` has been created

## docker-compose.yml

```yaml
version: '3'

volumes:
  html:

services:
  redis:
    image: redis:alpine
    container_name: nextcloud-cache
    restart: unless-stopped
    healthcheck:
     test: redis-cli ping || exit 1

  db:
    image: mariadb:jammy
    container_name: nextcloud:alpine
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW --innodb-read-only-compressed=OFF
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    environment:
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud_usr
      - MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_password
      - MYSQL_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password

  app:
    image: nextcloud:fpm-alpine
    container_name: nextcloud-app
    restart: unless-stopped
    volumes:
      - html:/var/www/html
    environment:
      - REDIS_HOST=redis
      - NC_default_phone_region=IT
      - MYSQL_HOST=db
      - MYSQL_PORT=3307
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud_usr
      - MYSQL_PASSWORD_FILE=/run/secrets/db_password
      - NEXTCLOUD_TRUSTED_DOMAINS=${HOST_NAME} ${HOST_IP}
      - TRUSTED_PROXIES=${HOST_NAME} ${HOST_IP}
      - OVERWRITEHOST=${HOST_NAME}
      - OVERWRITEPROTOCOL=https
      - SMTP_HOST=your.smtp
      - SMTP_NAME=your.email@your.provider
      - SMTP_SECURE=ssl
      - SMTP_PASSWORD_FILE=/run/secrets/smtp_password
    secrets:
      - db_password
      - smtp_password
    depends_on:
      redis:
        condition: service_healthy
      db:
        condition: service_started
    healthcheck:
      test: su -p www-data -s /bin/sh -c "php /var/www/html/occ status" | grep productname || exit 1

  web:
    image: nginx:alpine
    container_name: nextcloud-web
    restart: unless-stopped
    ports:
      - ${WEB_PORT}:80
    links:
      - app
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    volumes_from:
      - app
    healthcheck:
      test: curl -f http://localhost:80/ || exit 1

  cron:
    image: nextcloud:fpm-alpine
    container_name: nextcloud-cron
    entrypoint: /cron.sh
    restart: unless-stopped
    volumes_from:
      - app
    depends_on:
      redis:
        condition: service_healthy
      db:
        condition: service_started

secrets:
  db_password:
    file: db_password.txt
  smtp_password:
    file: smtp_password.txt

```

## References

- [Docker image of Nextcloud](https://github.com/nextcloud/docker)
- [Synology: How to Install Nextcloud Using Docker](https://mariushosting.com/synology-how-to-install-nextcloud-using-docker/)
