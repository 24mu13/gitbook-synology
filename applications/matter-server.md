# Matter Server

{% hint style="warning" %}
Theoretically this guide should work, but as for now [Matter Server](https://github.com/home-assistant-libs/python-matter-server) is not supported on Docker container!
See paragraph below for more details.
{% endhint %}

- Create folder `/volume1/docker/matter`
- Execute on shell: `ln /var/run/docker.sock /volume1/docker/docker.sock -s`
- **Container Manager**
- Execute on shell: `docker pull ghcr.io/home-assistant-libs/python-matter-server:stable`
- Create new container **matter-server**
    - Enable *auto-restart*
    - *Network*: **host**
    - *Ports Settings*:
        - Set local port for 5580 on container, e.g. 5580
    - *Volume Settings*:
        - /docker/matter -> /data
        - /docker/docker.sock -> /var/run/docker.sock [^1]

[^1]: In case you can't find the symbolic link, create the container, export it as JSON, change it adding the map, and import it again.

## Issues

### Host network

Matter Server requires to be run on the **host** network, otherwise you get the **Error 0x02000063** when starting Matter.

### IPv6

Matter Server requires **IPv6** to work properly, but Synology DSM does not support it on Docker containers!
I only managed to get it working for **bridge** network, using [Synology_ContainerManager_IPv6](https://github.com/007revad/Synology_ContainerManager_IPv6) script.

You will get so the **Error 0x02000065** when starting Matter.

## Assumptions

- You have installed **Container Manager**
- The folder `/volume1/docker` has been created
- You know how to access to the DSM shell

## References

- [Come attivare il supporto allo standard Matter su Home Assistant (Italian)](https://indomus.it/guide/come-attivare-il-supporto-a-matter-su-home-assistant/#docker)
