# Shlink

- Create new image from **shlinkio/shlink:stable**
- Create new container **shlink**
    - Enable *auto-restart*
    - *Ports Settings*:
        - Set a local port for 8080, e.g. 8080 or 8888
    - *Environment*:
        - DEFAULT_DOMAIN=your.domain
        - IS_HTTPS_ENABLED=false
        - GEOLITE_LICENSE_KEY=yourkey
        - If you need to use your NAS DBMS (e.g. MariaDB), remember to allow access from bridge subnet (i.e. 172.17.0.0) and add the following:
            - DB_DRIVER=mariadb
            - DB_USER=shlink_usr
            - DB_PASSWORD='yourpassword'
            - DB_HOST=192.168.1.1
            - DB_PORT=3307
- Control Panel -> Login Portal -> Advanced -> Reverse Proxy
- Create a new proxy for Nextcloud, e.g. https://your.domain -> http://192.168.1.1:8080
- For accessing UI app.shlink.io, execute on shell `sudo docker exec -it shlink shlink api-key:generate` to get an API key

The instructions above correspond to run the following commands:

```
docker run \
    --name shlink \
    --restart unless-stopped \
    -p 8081:8080 \
    -e DEFAULT_DOMAIN=your.domain \
    -e IS_HTTPS_ENABLED=false \
    -e GEOLITE_LICENSE_KEY=yourkey \
    -e DB_DRIVER=mariadb \
    -e DB_USER=shlink_usr \
    -e DB_PASSWORD='yourpassword' \
    -e DB_HOST=192.168.1.1 \
    -e DB_PORT=3307 \
shlinkio/shlink:stable

docker update --restart unless-stopped shlink
```
