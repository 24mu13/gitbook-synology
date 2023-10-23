# Portainer

- (Assuming `/volume1/docker` has been created together with *Container Manager*)
- Execute on shell: `ln /var/run/docker.sock /volume1/docker/docker.sock -s`
- Create new image from **portainer/portainer-ce:latest**
- Create new container **portainer**
    - Enable *auto-restart*
    - *Ports Settings*:
        - At least set one local port for 9000 on container, e.g. 9000 or 8080
    - *Volume Settings*:
        - (/docker/portainer -> /data)
        - /docker/docker.sock -> /var/run/docker.sock[^1]
- Reverse Proxy: create a corresponding entry for Portainer
- Create an account & log in Portainer
- Edit the local environment, assigning the LAN IP as public IP

[^1]: In case you can't find the symbolic link, create the container, export it as JSON, change it adding the map, and import it again.

## References

- [Synology: 30 Second Portainer Install Using Task Scheduler & Docker](https://mariushosting.com/synology-30-second-portainer-install-using-task-scheduler-docker/)
