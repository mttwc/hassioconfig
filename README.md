# HassioConfig

These instructions are for installing Home Assistant OS as a VM on Proxmox VE (PVE). Proxmox is a type-1 hypervisor that offers the ability to manage both VMs and Linux containers (LXCs) with ease.

## Why HA OS on Proxmox VE?

- HA OS is simple, has long-term support, and manages all of our OS updates in a way that doesn't break HA
- Proxmox VE allows us to install and manage other VMs and LXCs for functions that we do not wish to couple to HA
- Proxmox VE allows us to take snapshots of VMs, which we can use to restore HA if we break it after an update.
- While HA has snapshots, a broken HA means we need to set it up from scratch again before restoring the snapshot, and from experience, it's not a completely full restore either - we'd need to manually check that everything is still functioning.

## Why not Supervised?

I tried that for a number of years, but found it to be very difficult to maintain. Since it was an unsupported installation, HA had a lot of trouble updating both HA Core and addons. Things would break randomly, even when not updating HA itself. I originally tried Supervised to get the benefit of installing other docker containers side-by-side with HA, and we can get the same benefit with the Proxmox setup with the benefit of running an LTS.

The old Supervised documentation can be found [here](docs/SUPERVISED.md)

## Installation instructions

### Steps

The video instructions can be found here: https://www.youtube.com/watch?v=PrKQkI53xys

Additionally, the following would need to be done:

- Disable Secure Boot in the VM itself
- Enable the QEMU Guest Agent in the VM's Options in Proxmox