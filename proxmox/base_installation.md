# Proxmox Installation

This page documents how to install Proxmox based on Debian 12 - see [guide](../debian/base_installation.md) - using the official Proxmox [documentation](https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm).

## Installation

Make sure a static IP is defined for the machine with:

```bash
hostname --ip-address
```

If you do not see a localhost address everything is in order, however it is possible to make sure a static assignment by auditing the configuration `/etc/hosts`.

Install Proxmox with:

```bash
echo "deb [arch=amd64] http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list
wget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
apt update && apt full-upgrade -y
apt install -y proxmox-default-kernel
update-initramfs -tuck all && update-grub
reboot
```

After rebootig into Proxmox kernel, install the system with:

```bash
apt install -y proxmox-ve postfix open-iscsi chrony
apt remove -y linux-image-amd64 'linux-image-6.1*'
apt remove -y os-prober
update-initramfs -tuck all && update-grub
reboot
```

While postfix is installing a prompt will apear to setup the email server `local only` should be selected and the default mail server name should be used - just skip these options.

## Finish Installation

Cleanup after install with the useful scripts - [link](https://tteck.github.io/Proxmox/):

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
```

