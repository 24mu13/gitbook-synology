# Matter Server

- Create folder `/volume1/docker/matter`
- Execute on shell: `ln /var/run/docker.sock /volume1/docker/docker.sock -s`
- **Container Manager**
- Execute on shell: `docker pull ghcr.io/home-assistant-libs/python-matter-server:stable`
- Create new container **matter-server**
    - Enable *auto-restart*
    - *Ports Settings*:
        - Set local port for 5580 on container, e.g. 5580
    - *Volume Settings*:
        - /docker/matter -> /data
        - /docker/docker.sock -> /var/run/docker.sock [^1]

[^1]: In case you can't find the symbolic link, create the container, export it as JSON, change it adding the map, and import it again.

## Assumptions

- You have installed **Container Manager**
- The folder `/volume1/docker` has been created
- You know how to access to the DSM shell

## References

- [Come attivare il supporto allo standard Matter su Home Assistant (Italian)](https://indomus.it/guide/come-attivare-il-supporto-a-matter-su-home-assistant/#docker)
