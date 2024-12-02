# Migrate drives

## HDD to SDD

- List drives name
  ```shell
  sudo fdisk -l /dev/sata[1-9]
  ```
- Identify source and destination (e.g. /dev/sata1, /dev/sata2)
- Clone bit by bit using the block information got from command above (e.g. 512 byte): [^1]
  ```shell
  dd if=/dev/sata1 of=/dev/sata2 bs=512 conv=noerror,sync status=progress
  ```

## Remove from an SHR/RAID1 array

**Note**: _SHR_ is just an extra layer (LVM) on top of standard Linux-RAID (MD); 
so the commands below are simply based on this last one.

- Let's say you have 2 drives in SHR/RAID1
- List all the RAID arrays and volumes
  ```shell
  mdadm --detail /dev/md*
  ```
- Storage Manager -> Deactivate one of them (e.g. `/dev/sata2`); otherwise you can also do the following:
  - For each RAID array (e.g. `/dev/md0`, `/dev/md1`, `/dev/md2`)
    - Fail and remove the drive from that array, e.g.:
      ```shell
      mdadm /dev/md0 --fail /dev/sata2p1
      mdadm /dev/md0 --remove /dev/sata2p1
      wipefs -a /dev/sata2p1
      ```
- For each RAID array (e.g. `/dev/md0`, `/dev/md1`, `/dev/md2`)
  - Resize that array to just 1 drive, e.g:
    ```shell
    mdadm --grow --force --raid-devices=1 /dev/md0
    ```

## Convert ext4 to btrfs

The only supported way to convert an ext4 volume to btrfs is to backup the data, delete the volume, create a new btrfs volume, and restore the data. See the KB article in the references below.

### Backup & Restore

- Stop **Container Manager**
- Backup _Folder and Packages_ to C2 (12 hours for 500 GB, 20 MB/s)
  - **Important**: if you back up the entire system instead, the volume will be recreated as ext4
  - You need a C2 account and a subscription, the **trial** is fine
- Delete the current ext4 volume
  - You may need to completely uninstall **Container Manager**
- Re-create the volume with btrfs
- Restore _Folder and Packages_ (3 hours for 500 GB, 100 MB/s)
  - No need to restore system configurations
  - **Important**: MariaDB had disabled TCP port and reset to default for me, after restore

### In-place conversion

Ideally this should be the best & fastest option.

- Plug the ext4 drive into a Linux system with **mdadm** package installed [^2]
- Get the ext4 partition name, i.e. using **partitionmanager** (e.g. `/dev/vg1000/lv`) 
- `sudo e2fsck -fvy /dev/vg1000/lv`
- `sudo btrfs-convert /dev/vg1000/lv`
- (Optional) remove the ext2 backup image & balance, see documentation

Unfortunately at the time being, you get the following error, most probably because of a [bug](https://www.spinics.net/lists/linux-btrfs/msg142605.html) on large size partitions..

```
[...]
Create btrfs metadata
ERROR: failed to copy ext2 inode 12: -95
ERROR: error during copy_inodes -95
WARNING: error during conversion, the original filesystem is not modified
```

See also: [btrfs-convert](https://btrfs.readthedocs.io/en/latest/Convert.html)

## References
- [KB: How do I change an ext4 volume to a Btrfs volume?](https://kb.synology.com/en-my/DSM/tutorial/How_to_change_from_ext4_volume_to_btrfs_volume)
- [Remove drive from soft RAID](https://unix.stackexchange.com/questions/332061/remove-drive-from-soft-raid)
- [Can I reduce the number of drives in an existing storage pool or replace the drives with smaller capacity ones?](https://kb.synology.com/en-br/DSM/tutorial/Reduce_RAID_drives)

[^1]: Setting a bigger number (e.g. 64K) should make the process faster but you could encounter an error like `No space left on target`

[^2]: Even if available on [optware](https://github.com/Entware/Entware/wiki/Install-on-Synology-NAS) to be used directly inside the NAS, the package **mdadm** does not contain the _convert_ command!
