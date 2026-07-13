# LinkStack

{% hint style="warning" %}
In principle, you should be able to set root user (0:0) for the struckthrough steps, 
but apparently this setting is not available in Container Manager as per today.
Instead, by using the commands below, you will achieve same result.
{% endhint %}

- Run the following command as root/sudoer
    ```
    docker volume create linkstack
    
    docker run --detach \
        --name linkstack \
        --restart unless-stopped \
        --user 0:0 \
        --mount source=linkstack,target=/htdocs \
        -p 8190:443 \
        -e TZ=Europe/Rome \
        linkstackorg/linkstack:latest
    ```
- ~~**Container Manager**~~
- ~~Create new image from **linkstackorg/linkstack:latest**~~
- ~~Create new container **linkstack**~~
    - ~~Enable *auto-restart*~~
    - ~~*Ports Settings*:~~
        - ~~Set a local port for 443, e.g. 8190 (host port) → 443 (container port)~~
    - ~~*Volume Settings* (optional, for persistence):~~
        - ~~Mount container path `/htdocs` to a NAS shared folder (e.g. `/volume1/docker/linkstack`)~~
    - ~~*Environment*~~:
        - ~~TZ=Europe/Rome (or your timezone)~~
- Control Panel → Login Portal → Advanced → Reverse Proxy
- Create a new proxy for LinkStack root path, e.g. https://your.domain → https://127.0.0.1:8190

## Assumptions

- You have installed **Container Manager**
- You have a shared folder available (e.g. `/volume1/docker/linkstack`)
- You know how to access the DSM shell

## References

- [How to Install LinkStack on Your Synology NAS](https://mariushosting.com/how-to-install-linkstack-on-your-synology-nas/)