---
title: Configure networking and storage for a Proxmox homelab
date: 2023-01-25 09:00:00 -0000
categories: [Tech,Blog]
tags: [dell,server,homelab,storage,networking,fibre,10gb,optics,sfp,sfp+]
---
# Configuring my new host
With using proxmox as my underlying hyprevisor, there are a few steps to take to improve upon my old configuration as well as trying to optimise and improve the efficiency and fault tolerance of the setup.
- Make ZFS Raid 1 for installation of proxmox (Mirror the SSD's for fault tolerance)
- Setup Networking for remote storage (Direct Connect servers - bypass networking to alleivate resource usage)
- Setup Networking for management (Configure switchports and networking on proxmox)
- Setup Networking for VM consumption (Configure 10Gb ports for access to VM's)

## Disk config from a host level
I utilise a PERC H730-Mini in my dell server, this supports both hardware RAID (battery backed up) and also HBA Mode (for disk passthrough).  
The PERC initally came configured as RAID which meant that the PERC Mini would handle the disk configuration. This is both a benefit and negative. The benefit of this is that the RAID configuration does not need to be done on a software level, alleviating core resources. However the major downside to this is if the PERC Mini fails, then I lose all the RAID data.  

Therefore I wanted to configure the PERC Mini to be in HBA mode so the OS could handle the software RAID. With using the PERC Mini in HBA mode it does mean I am free to move the drives between systems as well, as all RAID config is stored indepedant of system.
Changing the H730-Mini to HBA mode was a very simple task (and much easier than previous cards). I just needed to go into the config utility from the BIOS after booting the server and selecting change mode. No extra firmware flashing was required.

## Tackling the Installation Storage config
For the OS installation, I set my two Kioxia Excercia SSD's in a ZFS RAID 1 config. This was for two reasons:
- If one of the SSDs fail, then I am fault tolerant and the system will continue to function, enabling me to replace the drive without losing config/data
- Creating a mirror means for any local VMs/LXC Containers on the system they will be more performant than if they were just running off a single SSD. Although I halve my storage capacity by doing this, I am in a better position with my core virtualisation system.

With this in mind I also wanted to look at using ZFS over BTRFS as my filesystem of choice. While both file systems support snapshots, my main storage server is truenas and this operates using ZFS. Because of this, I can utilise native ZFS commands such as ZFS Send for snapshot backups.

## Setup networking for remote storage
Both my Truenas and Proxmox server have dual 10GB SFP+ NICs installed. To take away the storage traffic from the network, I direct connected the two servers together. Using a /30 network to give me a point-to-point network and carved from my allocated `transit` /24 prefix.
With this, I enabled the Truenas system to listen on the new IP address for any NFS traffic to enable the Promox server to mount the shares. A couple of ACL updates later and the system is able to mount the shares away from the network switch.

If I need to upgrade the switch during this process as well, I know I'm free to do so without first having to shutdown all my VM's and LXC's on the hypervisor as the config doesn't touch this. The only impact will be for the VM's themselves and their connectivity to the backend storage - some of which utilise the switch to mount additional shares - of which these will be noted and powered off first.

## Setup networking for VM consumption
I have a Cisco 3650-X as my core switch, this is a 48 port POE+ and 10GB capable (provided I have the module installed - which I do). The module unfortunately only contains 2 10GB NICs. Becuase I want to ensure that my truenas server and proxmox server are on the network at 10GB speeds, I used a link from each to connect them to the network. This meant that
