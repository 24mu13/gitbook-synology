# Share NFS folder

## NAS

- Control Panel -> File Services -> NFS 
- Enable NFS service
- Control Panel -> Shared Folder
- Select the shared folder that you want to access with your NFS
- Permission -> Everyone -> Full Control (??)
- NFS Permissions -> Create
  - Hostname or IP: e.g. 192.168.0.0/255.255.255.0
  - Privilege: Read/Write
  - Squash: No mapping
  - Security: sys
  - Allow users to access mounted subfolders

## Linux Client

- Create the folder `/mnt/myfolder` 
- Add the following line to `/etc/fstab` (e.g assuming `192.168.0.1` for your NAS) [^1]
    ```
    192.168.0.1:/volume1/myfolder       /mnt/myfolder   nfs     defaults        0   0
    ```
- `mount /mnt/myfolder` 

[^1]: Mounting by meaning of the plain command, without editing `/etc/fstab`, did not work for me.

## References

- [How to access files on Synology NAS within the local network (NFS)](https://kb.synology.com/en-ca/DSM/tutorial/How_to_access_files_on_Synology_NAS_within_the_local_network_NFS)
- [Mount a synology folder as an NFS share under Linux](https://theawesomegarage.com/blog/mount-a-synology-folder-as-an-nfs-share-under-linux)
 