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

