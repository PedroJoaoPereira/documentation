# Create Pfsense VM in Proxmox

This page documents how to create a Pfsense VM in Proxmox just for personal notes and future reference.

## Create VM

Clicking on the `proxmox-server` node and then on `Create VM` a new prompt will be show and the VM should have the following specifications:

---

General:

- Node: `proxmox-server`
- VM ID: `100`
- Name: `pfsense-router`

- Start at boot: `true`
- Start/Shutdown order: `1`
- Tags: `network`, `router`

---

OS:

- ISO image: `pfsense iso`
- Type: `other`

---

System:

- Machine: `q35`

---

Disks:

- Bus/Device: `VirtIO Block`
- Disk size: `16GB`
- Discard: `true`

---

CPU:

- Cores: `4`
- Type: `host`

- AES: `on`

---

Memory:

- Memory: `4096`

- Ballooning Device: `false`

---

Network:

- No network device: `true`

---

## After Creation

After creation, by clicking in the machine, it is possible to validate all the above options and there is the possibility to remove `Use tablet for pointer` to `No` to save a bit on CPU.

Additionally, the PCI passthrough NIC devices should be added by resorting to the [documentation](../enable_iommu_and_passthrough.md#finalizing-configuration).

The VM can be started and the default installation can go through.

## After Installation

After the basis OS installation the CD-ROM disk drive with the ISO can be removed.



