# Prepare Proxmox Disks for Storage

This page documents how to setup storages for Proxmox in different partitions just for personal notes and future reference.

## Partition Disks

Open Proxmox web-ui, click the `proxmox-server` node and then the `shell` option. The terminal will show up with a connection established with the node. Partition disks according to the following schema:

The `/dev/nvme0n1` disk should have a new partition:
- `/dev/nvme0n1p5` size _All Available_ used for `VM data`

The `/dev/sda` disk should have the following partitions:
- `/dev/sda1` size _64GB_ used for `ISO storage` mounted at `/proxmox/isos`
- `/dev/sda2` size _256GB_ used for `VM Backup storage` mounted at `/proxmox/backups`
- `/dev/sda3` size _8GB_ used for `Cold storage` mounted at `/proxmox/cold`

Partition the disks with `fdisk /dev/<DISK>` and create the appropriate filesystems and mounts:

```bash
# create ext4 filesystem in the devices
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3

# create mount point directories
mkdir -p /proxmox/isos /proxmox/backups /proxmox/cold

# mount the devices at the target directories
mount /dev/sda1 /proxmox/isos
mount /dev/sda2 /proxmox/backups
mount /dev/sda3 /proxmox/cold

# add mounted points to fstab
cat >> /etc/fstab << EOF
# sda1 - /proxmox/isos - ISO storage
UUID=ccf45c70-3ba5-405e-a56b-d064eeaed35d       /proxmox/isos           ext4        defaults 0 2
# sda2 - /proxmox/backups - VM data backups storage
UUID=fc55ced7-c558-4deb-aa00-aa5e978830f4       /proxmox/backups        ext4        defaults 0 2
# sda3 - /proxmox/cold - cold Proxmox storage
UUID=c86675be-68fe-48be-abc1-90c187b3f1ef       /proxmox/cold           ext4        defaults 0 2

# ---

EOF

reboot
```

The devices can be displayed with:

```bash
lsblk -o name,uuid,size
```

The mount points can be validated with

```bash
df -h
```

## Setup Proxmox Storage

Add the `ZFS` filesystem to `/dev/nvme0n1p5` by clicking the `proxmox-server` node, the option `Disks`, `ZFS`, `Create: ZFS` with the name `aa-local-data-zfs`. After setting this storage and using Proxmox shell it is possible to disable atime records to get a bit more performance of the zfs pool by running the command:

```bash
zfs set atime=off aa-local-data-zfs
zfs get atime
```

And the zfs pool should now be listed as atime value off.

Now with the disks properly partitioned and setup, add the storages for all disks by clicking the `Datacenter` option on the tree view, `Storage` in the datacenter options and adding:

- `ZFS` storage with the ID `aa-local-data-zfs` - this storage should already be added due to the actions above and set to content: `Disk Image, Container`
- `Directory` storage with the ID `ba-extra-backups-ext4` and the directory `/proxmox/backups/` - content: `VZDump backup file`
- `Directory` storage with the ID `ia-extra-isos-ext4` and the directory `/proxmox/isos/` - content: `ISO image`
- `Directory` storage with the ID `za-extra-cold-ext4` and the directory `/proxmox/cold/` - content: `Container template, Snippets`

Disable the `local` storage as it should be reserved for the Proxmox host only.

