# Container Manager

Container Manager unites status monitoring, container image management, and resource management in a straightforward interface.

For more details see the [product page](https://www.synology.com/en-global/dsm/feature/container-manager).

## Firewall

When using **bridge** network, you need to open traffic for those addresses. Few scenarios e.g.:
- Internet, i.e. your container accessing Internet
- DBMS, i.e. your container trying to reach a DB

## Network mode in _Docker Compose_

**compose.yaml**:
```yml
services:
  app:
    network_mode: bridge|host
```

