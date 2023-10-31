# Home Assistant

- Create folder `/volume1/docker/homeassistant`
- **Container Manager**
- Create new image from **homeassistant/home-assistant:latest**
- Create new container **home-assistant**
  - Enable *auto-restart*
  - *Network*: **host**
  - *Volume Settings*:
      - /docker/homeassistant -> /data
- **Home Assistant**
- Create an account & login

## Assumptions

- You have installed **Container Manager**
- The folder `/volume1/docker` has been created

## References

- [Synology: Install Home Assistant With Portainer](https://mariushosting.com/synology-install-home-assistant-with-portainer/)
