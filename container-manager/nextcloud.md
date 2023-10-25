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
  cache:
  html:

services:
  redis:
    image: redis:alpine
    container_name: nextcloud-cache
    restart: unless-stopped
    volumes:
      - cache:/data
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

### Variant: MariaDB on DSM

You can use of course the regular _Synology_ package for MariaDB.

In this case just remove the **db** service/section in the **docker-compose.yml** file above and set accordingly MYSQL_HOST variable.

### Variant: Shared folder on DSM

You can use a regular folder on your NAS.

- Control Panel -> Shared Folders
- Create new **nextcloud** shared folder
- On tab *General*, check only "Hide this shared folder..."
- Set permissions for **www-data** user/group (ID 82 in _alpine_ images) to that folder (e.g. `/volume1/nextcloud/data`)
  ```shell
  sudo chown 82:82 -R /volume1/nextcloud/data
  sudo chmod 0770 -R /volume1/nextcloud/data
  ``` 
- Add the proper mapping to the app service in **docker-compose.yml**
  ```yaml
  app:
    image: nextcloud:fpm-alpine
    container_name: nextcloud-app
    restart: unless-stopped
    volumes:
      - html:/var/www/html
      - /volume1/nextcloud/data:/var/www/html/data:rw
    # etc ...
  ```

## Migrate existing installation

The following instructions are based on official documentation referenced here below and some practical experience. 

1. Prepare a brand new stack of Nextcloud (i.e. **destination**), by also _adding the admin user_
2. (opt?) Stop it
3. From the source stack of Nextcloud
   - Copy or move `html/config/config.php`
   - Copy or move `html/data/`, and make sure user **www-data** has 0770 permissions
   - Copy or move **DB**
4. Start the destination again

### Backup and restore MariaDB

- Backup
  ```shell  
  mysqldump -u root -p{yourpassword} nextcloud > {dumpfile}
  ```
- Restore 
  ```shell  
  mysql --user root --password nextcloud < {dumpfile}
  ```
- Running commands from DSM shell
  ```shell
  sudo /volume1/@appstore/MariaDB10/usr/local/mariadb10/bin/mysql...   
  ```  
- Running commands from container
  ```shell
  sudo docker exec nextcloud-db sh -c "mysql..."   
  ```  

### Permissions on data folder

- Data folder: `/var/www/html/data`
- Owner: www-data/www-data
- Permissions: 0770

Depending of used architecture in containers, you should use **www-data** internal ID instead of user/group names.
- Apache: 33
- FPM/nginx: 82

## References

- [Docker image of Nextcloud](https://github.com/nextcloud/docker)
- [Synology: How to Install Nextcloud Using Docker](https://mariushosting.com/synology-how-to-install-nextcloud-using-docker/)
- [Migrating to a different server](https://docs.nextcloud.com/server/latest/admin_manual/maintenance/migrating.html)
