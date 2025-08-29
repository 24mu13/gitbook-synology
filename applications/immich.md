# Immich

- Create folder `/volume1/immich` as library for photos/videos - this allows to set a quota for example
  - Give **read & write** permissions to user `ContainerManager`
- Create folder `/volume1/docker/immich` and the following inside:
  - **db** folder 
  - [compose.yaml](https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml)
  - [.env](https://github.com/immich-app/immich/releases/latest/download/example.env), set
    - UPLOAD_LOCATION=`/volume1/immich`
    - DB_DATA_LOCATION=`./db`
    - TZ=`Your/Timezone` (e.g. Europe/Rome)
- **Container Manager**
- Create new project **immich** by using the compose file above
- Control Panel -> Login Portal -> Advanced -> **Reverse Proxy**
- Create a new proxy for Immich, e.g. https://your.domain -> http://192.168.1.1:2283
- Custom Header -> Create -> Web Socket 
- Enjoy **Immich**

## References

- [Install > Synology [Community]](https://immich.app/docs/install/synology/)