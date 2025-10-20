# Home Assistant

{% hint style="warning" %}
This guide is for **Home Assistant Core**. If you want to install **Home Assistant Supervised** on Synology, please refer to the [Virtual Machines Manager](/vmm/vmm-home-assistant.md) section.
{% endhint %}

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
- For using **Matter** devices with Home Assistant, you also need to install **Matter Server** from [Synology Community](https://synocommunity.com) packages.

## References

- [Synology: Install Home Assistant With Portainer](https://mariushosting.com/synology-install-home-assistant-with-portainer/)
