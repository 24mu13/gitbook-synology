# Watchtower

{% hint style="warning" %}
Usage of Watchtower is recommended only because Synology does not provide a feature to 
auto-update containers, neither a simple notification for projects or images with tags
other than latest, when a new image is available. 
See [this discussion](https://community.synology.com/enu/forum/1/post/161251) for more details.
{% endhint %}

- Create new image from **nickfedor/watchtower**[^1]
- Create new container **watchtower**
    - Enable *auto-restart*
    - *Environment*:
        - WATCHTOWER_CLEANUP=true
        - WATCHTOWER_SCHEDULE=0 0 2 * * *
    - *Volume Settings*:
        - /docker/docker.sock -> /var/run/docker.sock [^2]      

[^1]: The original [watchtower](https://github.com/containrrr/watchtower) is basically discontinued, while [beatkind](https://github.com/beatkind/watchtower) fork is requiring an higher Docker API version, at the time being.

[^2]: In case you can't find the symbolic link, create the container, export it as JSON, change it adding the map, and import it again.

## Skip certain containers

### Projects

You can skip certain containers (services) by adding the label `com.centurylinklabs.watchtower.enable=false` to the container you want to skip.

Here below an example:

```yaml
services:
  redis:
    image: redis:alpine
    container_name: nextcloud-cache
    restart: unless-stopped
    volumes:
      - cache:/data
    healthcheck:
      test: redis-cli ping || exit 1
    labels:
      - com.centurylinklabs.watchtower.enable=false
```

## Assumptions

- You have installed **Container Manager**
- The folder `/volume1/docker` has been created

## References

- [Watchtower – Automated Updates in Container Manager on a Synology NAS](https://drfrankenstein.co.uk/watchtower-automated-updates-in-container-manager-on-a-synology-nas/)