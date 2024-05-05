# Debian Installation

This page documents how to install base Debian OS with UEFI just for personal notes and future reference.

Assuming only four partitions for UEFI, boot, host and swap. The OS will be installed with a single root account - which is not complying with some secutiry procedures recommended. Additionally, a swap partition will be added as it can be used to optimize RAM usage.

## Partitioning Schema

The disk will contain a GPT partition table with four partitions:
- `/dev/nvme0n1p1` size _512MB_ - `EFI System Partition` mounted at `/boot/efi`
- `/dev/nvme0n1p2` size _512MB_ - `Boot Partition` mounted at `/boot`
- `/dev/nvme0n1p3` size _4GB_ - `Swap Partition` mounted at `swap`
- `/dev/nvme0n1p4` size _8GB_ - `Root Partition` mounted at `/`

## Installation

Using an installation media choose: `Advanced Options` and then `Expert Install`. After this selection the installation will start and everything should be selected accordding to the targetted preferences, in this case a static IP will be set in the system and, for that reason, at the time of the option `Configure Network` and after choosing the primary network interface, the option for manual network interface configuration was chosen.

The hostname for this machine will be `proxmox-server` and its domain `home-network`. For this installation no additional users will be setup other than the root user. When the partition disks screen is reached choose `Manual` partition of the disks according to the above schema. Choosing the filesystem `EXT4` for both `boot` and `root` partitions as EXT2 will be deprecated in the near future. When choosing this filesystem, the mounting options should include `noatime` to avoid unecessary writing operations to the disk as well as `discard` so the SSD can have a bit longer lifetime, in case of a HDD the default options will work.

While configuring the package manager choose the mirror option closer to the system and select only the essentials repositories by declining all extra prompted repositories / branches. When choosing the software to be installed also allow the backported software into the repositories, disable any desktop environment selected by default and enable SSH server - only the system essentials and ssh server should be selected.

### Create Insecure Connection

In order to work remotely with ssh while using a root account the ssh daemon should be configure to allow root login connections- this is dangerous but it will be enabled during a short period of time while configuring connection keys. To enable the root ssh logind edit the file:

```bash
nano /etc/ssh/sshd_config
```

Add the line:

```text
...
PermitRootLogin yes
...
```

Restart the ssh service with or reboot:

```bash
systemctl restart ssh
```

### Create Secure Connectio

Create SSH keys to be copied over to the new system (remote) and the root login can be disabled, by executing the following on the machine that will connect to the remote system:

```bash
ssh-keygen
```

Then copy the ssh pub key over to the fresh new installation:

```bash
ssh-copy-id -i ~/.ssh/id_personal.pub  root@192.168.1.20
```

Configure the machine to use the correct ssh key pair when connecting to the remote system with:

```bash
nano ~/.ssh/config
```

Add the new section referring the the remote system:

```text
# proxmox-server
Host 192.168.1.20
  User              root
  IdentityFile      ~/.ssh/id_personal
```

Disable the root login option by reverting the `PermitRootLogin yes` at `nano /etc/ssh/sshd_config` and restart the ssh service with or reboot:

```bash
systemctl restart ssh
```

## Update Source Files and Dependencies

Update the sources file with the appropriate stable repositories and update / upgrade the system:

```bash
# update sources files
cat > /etc/apt/sources.list << EOF
# default debian 12 bookworm repositories
deb http://deb.debian.org/debian bookworm main
deb http://deb.debian.org/debian bookworm-updates main
deb http://deb.debian.org/debian bookworm-backports main
deb http://security.debian.org/debian-security bookworm-security main

EOF

# remove warning regarding firmware branch changes
su -c 'echo "APT::Get::Update::SourceListWarnings::NonFreeFirmware \"false\";" > /etc/apt/apt.conf.d/no-bookworm-firmware.conf'
# update the system
apt update && apt upgrade -y
# install dependencies
apt install -y neovim

# add custom alias to the bash shell
echo "" >> /etc/bash.bashrc
echo "# system wide - custom alias WIP" >> /etc/bash.bashrc
echo "alias ll='ls -la'" >> /etc/bash.bashrc
echo "alias v='nvim'" >> /etc/bash.bashrc
echo "" >> /etc/bash.bashrc
# update the bash shell session
source /etc/bash.bashrc
```

## Update GRUB Configuration

This will be changed so the boot happens faster and with less logging:

```bash
# update grub boot configurations
sed -i 's/GRUB_TIMEOUT=5/GRUB_TIMEOUT=1/g' /etc/default/grub
sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="quiet"/GRUB_CMDLINE_LINUX_DEFAULT="quiet loglevel=3 nowatchdog"/g' /etc/default/grub

# regenerate initramfs and grub configurations
update-initramfs -tuck all && update-grub

reboot
```

