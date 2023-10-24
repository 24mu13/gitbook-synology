# Portainer

- Execute on shell: `ln /var/run/docker.sock /volume1/docker/docker.sock -s`
- **Container Manager**
- Create new image from **portainer/portainer-ce:latest**
- Create new container **portainer**
    - Enable *auto-restart*
    - *Ports Settings*:
        - At least set one local port for 9000 on container, e.g. 9000 or 8080
    - *Volume Settings*:
        - (/docker/portainer -> /data)
        - /docker/docker.sock -> /var/run/docker.sock *[^1]
- Control Panel -> Login Portal -> Advanced -> **Reverse Proxy**
- Create a new proxy for Portainer, e.g. https://your.domain -> http://192.168.1.1:9000
- **Portainer**
- Create an account & login
- Edit the local environment, assigning the LAN IP (e.g. 192.168.1.1) as public IP

[^1]: In case you can't find the symbolic link, create the container, export it as JSON, change it adding the map, and import it again.

## Assumptions

- You have installed **Container Manager**
- The folder `/volume1/docker` has been created
- You know how to access to the DSM shell

## References

- [Synology: 30 Second Portainer Install Using Task Scheduler & Docker](https://mariushosting.com/synology-30-second-portainer-install-using-task-scheduler-docker/)
