# Nextcloud AIO

[Nextcloud All-in-One](https://github.com/nextcloud/all-in-one) (AIO) is the official way to run Nextcloud in Docker.
You start only one **master container**. It then creates and manages all the other containers (Nextcloud, Apache, PostgreSQL, Redis, Collabora, Talk, backup, ...).

Compared to the [compose based setup](nextcloud.md):

- No project (`compose.yaml`) to maintain
- Upgrades for Nextcloud and all its containers run from the AIO web interface, in the right order
- Built-in backup (BorgBackup), which can also update the containers automatically every day
- Downside: it is less flexible (see [Database](#database) below)

## Assumptions

- You have installed **Container Manager**
- SSH access to DSM is enabled (see [SSH](../connectivity/ssh.md)). The master container needs `--init` and `--sig-proxy=false`, so run it from the CLI and not from the Container Manager wizard.
- You have a domain (e.g. `your.domain`) that points to your NAS, and a valid certificate on DSM (see [SSL/TLS](../connectivity/ssl-tls.md))

## Setup

- Create the data folder (e.g. a hidden shared folder **nextcloud**, see [Variant: Shared folder on DSM](nextcloud.md#variant-shared-folder-on-dsm)):
  ```shell
  sudo mkdir -p /volume1/nextcloud/data
  ```
  You don't need to change ownership. AIO sets the permissions of the data folder by itself.
- Start the master container via SSH:
  ```shell
  sudo docker run \
    --init \
    --sig-proxy=false \
    --name nextcloud-aio-mastercontainer \
    --restart always \
    --publish 8080:8080 \
    --env APACHE_PORT=11000 \
    --env APACHE_IP_BINDING=0.0.0.0 \
    --env NEXTCLOUD_DATADIR="/volume1/nextcloud/data" \
    --env NEXTCLOUD_MOUNT="/volume1/" \
    --volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
    --volume /var/run/docker.sock:/var/run/docker.sock:ro \
    ghcr.io/nextcloud-releases/all-in-one:latest
  ```
- Control Panel -> Login Portal -> Advanced -> **Reverse Proxy** -> Create
  - Source: `HTTPS`, hostname `your.domain`, port `443`
  - Destination: `HTTP`, hostname `localhost`, port `11000`
  - Custom Header -> Create -> **WebSocket**
  - Advanced Settings: increase the timeouts (e.g. `3600`) so big uploads don't fail
- Open the AIO interface at `https://192.168.1.1:8080` (it uses a self-signed certificate). Write down the passphrase it shows you.
- Enter `your.domain`, choose the optional containers, set the timezone and click **Download and start containers**
- Enjoy **Nextcloud** at https://your.domain

### Notes

- **Don't change** the container name `nextcloud-aio-mastercontainer` or the volume name `nextcloud_aio_mastercontainer`. AIO expects these exact names.
- `NEXTCLOUD_DATADIR` only works for the **first** installation. Changing it afterwards needs a manual migration.
- Keep port `8080` reachable only on your LAN. It gives access to the whole AIO interface.
- If you use the firewall, allow the Docker subnets (see [Networking](README.md#networking)). Otherwise the domain check and the containers can't reach each other.
- Exclude all `nextcloud-aio-*` containers from [Watchtower](watchtower.md). Upgrades must go through AIO.

## Access directories on the host

AIO doesn't use a project, so you can't add volume mappings. To let Nextcloud read and write folders on the NAS (as described [here](https://github.com/nextcloud/all-in-one#how-to-allow-the-nextcloud-container-to-access-directories-on-the-host)):

- Set `NEXTCLOUD_MOUNT` on the master container (already included above):
  ```shell
  --env NEXTCLOUD_MOUNT="/volume1/"
  ```
  The folder is mounted into the Nextcloud container at the **same path**.
- If you add it later, recreate the master container (see [Change master container options](#change-master-container-options)). Then in the AIO interface, stop and start the containers.
- Give the container's `www-data` user (ID `33`) access to the folders you want to share, e.g.:
  ```shell
  sudo chown -R 33:33 /volume1/shared/photos
  sudo chmod -R 0770 /volume1/shared/photos
  ```
- In Nextcloud: Apps -> enable **External storage support**
- Administration settings -> External storage -> add a **Local** storage with path `/volume1/shared/photos`

**Note**: `NEXTCLOUD_DATADIR` and `NEXTCLOUD_MOUNT` must be different folders. Don't use the data folder (or a folder inside it) as external storage.

## Database

AIO always uses its own **PostgreSQL** container (`nextcloud-aio-database`). You **can't** use an external database, so the Synology **MariaDB** package can't be used.

The [user_sql](https://github.com/nextcloud/all-in-one#user_sql) paragraph is a different topic. It covers the _user_sql_ app, which logs users in from an external SQL table. It doesn't change where Nextcloud stores its data.

If you need MariaDB on DSM, use the [compose based setup](nextcloud.md#variant-mariadb-on-dsm).

## Migrate existing installation

Official guide: [migration.md](https://github.com/nextcloud/all-in-one/blob/main/migration.md). You have two options.

### Files only (simplest)

Create the users again in AIO and copy their files. You lose shares, calendars, contacts and app data.

1. Set up a new AIO instance and create the same users
2. Stop the containers in the AIO interface
3. Copy `data/<user>/files` from the old instance into `/volume1/nextcloud/data/<user>/files`
4. Start the containers and rescan:
   ```shell
   sudo docker exec --user www-data nextcloud-aio-nextcloud php occ files:scan --all
   ```

### Files and database (MariaDB -> PostgreSQL)

This keeps everything, but you have to convert the old MariaDB database to PostgreSQL first. In short:

1. Update the old instance to the **same major version** that AIO installs
2. Start a temporary PostgreSQL container and convert the old database into it:
   ```shell
   sudo docker exec --user www-data nextcloud-app php occ db:convert-type --all-apps --password "<pg_password>" pgsql <pg_user> <pg_host> <pg_database>
   ```
3. Create a dump of the PostgreSQL database with `pg_dump`
4. Follow the official guide to put the dump into the `nextcloud_aio_database_dump` volume as `database-dump.sql`. Then update `config.php` (database user and password, `datadirectory`, ...) and copy the data folder.
5. Start the containers from the AIO interface

**Note**: follow the official guide step by step, because the details change between AIO versions. Take a backup first (see [Backup and restore MariaDB](nextcloud.md#backup-and-restore-mariadb)).

## Maintenance

### Update

- AIO interface -> **Stop containers** -> **Start and update containers**
- If a new master container is available, the interface shows it. Click **Update mastercontainer** first.
- Or enable daily backup with **automatic updates** in the AIO interface

### Backup

- AIO interface -> Backup section -> set a target folder (e.g. `/volume1/backup/nextcloud-aio`) and create a backup
- Restore works from the same interface (also on a new installation)

### Change master container options

To add or change an `--env` option (e.g. `NEXTCLOUD_MOUNT`):

```shell
sudo docker stop nextcloud-aio-mastercontainer
sudo docker rm nextcloud-aio-mastercontainer
# run the same "docker run" command again, with the new options
```

The configuration is kept in the `nextcloud_aio_mastercontainer` volume.

## How to

Run `occ` commands in AIO like this (see also [Nextcloud CLI](../occ-cli.md)):

```shell
sudo docker exec --user www-data -it nextcloud-aio-nextcloud php occ <command>
```

### Recover data when seems lost

```shell
sudo docker exec --user www-data nextcloud-aio-nextcloud php occ files:scan --all
```

### Set default phone region

```shell
sudo docker exec --user www-data nextcloud-aio-nextcloud php occ config:system:set default_phone_region --value="IT"
```

### Configure SMTP

Administration settings -> Basic settings -> Email server (see also [Fix errors when configuring SMTP](nextcloud.md#fix-errors-when-configuring-smtp)).

## References

- [Nextcloud All-in-One](https://github.com/nextcloud/all-in-one)
- [AIO reverse proxy documentation](https://github.com/nextcloud/all-in-one/blob/main/reverse-proxy.md)
- [AIO migration documentation](https://github.com/nextcloud/all-in-one/blob/main/migration.md)

