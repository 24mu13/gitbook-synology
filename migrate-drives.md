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

At the time being I didn't manage to convert an ext4 drive entirely to btrfs. 
Feel free to raise a pull-request for any suggestion/idea.

### Backup & restore (entire system)

**Note**: this procedure is only theoretical, didn't worked!

- Backup the entire system to C2
- Power off and replace the old ext4 with a spare new drive
- Start the NAS and delete the old volume, create the new one with btrfs
- Restore the system (took 5 days for 2TB, e.g.)

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

- [Remove drive from soft RAID](https://unix.stackexchange.com/questions/332061/remove-drive-from-soft-raid)
- [Can I reduce the number of drives in an existing storage pool or replace the drives with smaller capacity ones?](https://kb.synology.com/en-br/DSM/tutorial/Reduce_RAID_drives)
- [How do I change an ext4 volume to a Btrfs volume?](https://kb.synology.com/en-my/DSM/tutorial/How_to_change_from_ext4_volume_to_btrfs_volume)

[^1]: Setting a bigger number (e.g. 64K) should make the process faster but you could encounter an error like `No space left on target`

[^2]: Even if available on [optware](https://github.com/Entware/Entware/wiki/Install-on-Synology-NAS) to be used directly inside the NAS, the package **mdadm** does not contain the _convert_ command!
