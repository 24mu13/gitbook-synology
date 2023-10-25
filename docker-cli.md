# B. Docker CLI

- Start/restart/reapply changes:
  ```
  docker compose up -d
  ```

- Restart a single service, e.g.:
  ```
  docker compose restart web
  ```

- Update a stack
  ```
  docker-compose pull
  docker compose up -d
  ```

- Stop & remove all
  ```
  docker compose down
  docker rm -f $(docker ps -a -q)
  docker volume rm $(docker volume ls -q)
  ```

- Delete unused images
  ```
  docker image prune -a
  ```

- Execute on container
  ```
  docker compose exec --user <user> <service> "command"
  ```

- Follow container logs
  ```
  docker compose logs <service> -f
  ```

- List all content in a volume (by using *busybox*)
  ```
  docker run --rm -i -v=volume_name:/tmp/ busybox find /tmp/
  ```

- Copy from/to volume (by using *busybox*)
  ```
  CID=$(docker run -d -v volume_name:/html busybox true)
  cp $(CID):/somepath ./somepath
  ```