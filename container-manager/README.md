# Container Manager

Container Manager unites status monitoring, container image management, and resource management in a straightforward interface.

For more details see the [product page](https://www.synology.com/en-global/dsm/feature/container-manager).

## Networking

If you set Firewall, remember to open all the traffic for the private subnet used by _Container Manager_.

Few scenarios e.g.:
- **Internet**
- **DBMS**, i.e. your container trying to reach a DB on host
- **DNS**, i.e. your container trying to identify another container in the same project
- **nginx**, i.e. expose your container using nginx

If you want to open the traffic for all subnet used typically by _Container Manager_, you could use the following subnet:
`172.16.0.0/255.240.0.0`

### Double check connectivity

- Open a shell inside container: `docker exec -it mycontainer sh`
- Ping google.com: `ping google.com`
- Ping/nslookup another container in the same subnet: `nslookup anothercontainer`

**Note**: manually install **bind9-dnsutils** package for _nslookup_ or **dnsutils-ping** for _ping_ if needed. 

## Docker commands

Start/restart/reapply changes:
`docker compose up -d`

Restart a single service, e.g.:
`docker compose restart web`

Update:
`docker-compose pull`
`docker compose up -d`

Stop & remove all
`docker compose down`
`docker rm -f $(docker ps -a -q)`
`docker volume rm $(docker volume ls -q)`

Delete unused images
`docker image prune -a`

Execute on container
`docker compose exec --user <user> <service> "command"`

Follow container logs
`docker compose logs <service> -f`

List all content in a volume (by using *busybox*)
`docker run --rm -i -v=volume_name:/tmp/ busybox find /tmp/`

Copy from/to volume (by using *busybox*)
```
CID=$(docker run -d -v volume_name:/html busybox true)
cp $(CID):/somepath ./somepath
```