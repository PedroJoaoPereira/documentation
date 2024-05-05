# Prepare Proxmox PCI Devices Passthrough

This page documents how to setup PCI passthrough for virtualization just for personal notes and future reference resorting to the official [documentation](https://pve.proxmox.com/wiki/PCI_Passthrough).

## Verify Requirements

To have PCI passthrough enabled the CPU / MoBo needs to be compatible and the motherboard needs to be configured with the appropriate features.

If any of these checks do not show the expected output it means the PCI passthrough was not correctly enabled in the system or is not supported.

Verify requirements with:

```bash
dmesg | grep -e DMAR -e IOMMU
# there should be a line that looks like "DMAR: IOMMU enabled"

dmesg | grep 'remapping'
# there should be one of the following lines
# AMD-Vi: Interrupt remapping enabled
# DMAR-IR: Enabled IRQ remapping in x2apic mode
```

Using Proxmox tools it is possible to display all the devices of a node and its corresponding IOMMU group.

```bash
pvesh get /nodes/proxmox-server/hardware/pci --pci-class-blacklist ""
```

It should be assured that the PCI device to be passthrough is not sharing a IOMMU group with another device as the kernel of the host system will not allow the device to be used by another system due to memory sharing. If the motherboard has support for `Acess Control Services (ACS)`, the feature when enabled will split the PCI devices between different groups. If the motherboard does not have this feature an override can be set in the kernel parameters - even though it is not recommended or secure, read more in the official [documentation](https://pve.proxmox.com/wiki/PCI_Passthrough).

## Enable IOMMU

To enable IOMMU in the system some additional parameters need to be added to the kernel options. It relevant that this guide is covering the setup for a AMD machine which is different from an Intel one.

Edit `/etc/default/grub` so the parameter `iommu=pt` is added to the `GRUB_CMDLINE_LINUX_DEFAULT`.

Edit `/etc/modules` file should also be edited to include the following kernel modules on load:

```text
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd #not needed if on kernel 6.2 or newer
```

Regenerate initramfs and grub configiration, followed by a reboot:

```bash
update-initramfs -tuck all && update-grub

reboot
```

Check if the modules are being loaded with:

```bash
lsmod | grep vfio

dmesg | grep -e DMAR -e IOMMU -e AMD-Vi
# should output the following line
# IOMMU, Directed I/O or Interrupt Remapping
```

## Host Configuration

In order to passthrough a PCI device, the device needs to be available and not in use by the host. Running the following command allows to display all the devices, driver and kernel modules in use. If the desired device does not have a vfio driver it means it is in use by the host and can never be passthrough:

```bash
lspci -nnk
```

Using the above command, the desired devices to be passthrough can be noted. Updating the initramfs hook scripts it is possible to force a vfio driver to load on the device by creating a script, however the device path needs to be found by its Id with:

```bash
find /sys/devices/pci* | grep 0c:00.0 | grep driver_override
```

Being `0c:00.0` the device ID listed by `lspci -nnk`, with this the script to update initramfs with can be created:

```bash
cat > /etc/initramfs-tools/scripts/init-top/bind_vfio.sh << EOF
#!/bin/sh -e

# force each of the passthrough devices to use vfio driver
echo "vfio-pci"  > /sys/devices/pci0000:00/0000:00:03.2/0000:0c:00.0/driver_override
echo "vfio-pci"  > /sys/devices/pci0000:00/0000:00:03.2/0000:0c:00.1/driver_override

# load driver into devices
modprobe -i vfio-pci

EOF

# give root permissions to script and make it executable
chmod 755 /etc/initramfs-tools/scripts/init-top/bind_vfio.sh
chown root:root /etc/initramfs-tools/scripts/init-top/bind_vfio.sh

# use vfio module on initramfs
echo "vfio-pci" >> /etc/initramfs-tools/modules

update-initramfs -tuck all && update-grub

reboot
```

Validation can be done using the device ID and the following command:

```bash
lspci -s 0c:00.0 -v
```

Everything is working as expected if the device is using the driver `vfio-pci`. It is possible to read more about this operation [here](https://forum.proxmox.com/threads/pci-passthrough-selection-with-identical-devices.63042/).

## Finalizing Configuration

Now the device can be added with: WIP WIP WIP

