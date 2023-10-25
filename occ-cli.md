### C. Nextcloud CLI (occ)

All the following commands are executable also through the container.
Assuming your application container is **nextcloud-app**, for instance, you should run the following from the DSM shell.

```shell
docker exec --user www-data nextcloud-app php occ <ommand here>
```

- Scan & cleanup
  ```
  occ files:scan --all
  occ files:cleanup
  ```

- Set maintenance mode
  ```
  occ maintenance:mode --[on|off]
  ```

- Check setup
  ```
  occ integrity:check-core
  ```

- Upgrade to a newer version (no need using containers)
  ```
  occ upgrade
  ```