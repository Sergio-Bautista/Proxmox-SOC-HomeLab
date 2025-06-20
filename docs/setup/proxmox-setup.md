Proxmox VE Server Setup


Hardware Preparation: (e.g., BIOS settings, boot order for USB/DVD). Ensure your Proxmox host has at least one physical network interface card (NIC).

Proxmox VE Installation:
Download Proxmox ISO [https://www.proxmox.com/en/downloads].
Create bootable USB. [Using rufus or any other software]
Boot from USB and follow installer.
Network configuration during install (configure eno1 and vmbr0 for Proxmox management, as described below).
Initial login to Proxmox web UI (https://[your_proxmox_ip]:8006).

Storage Configuration:
Details of your storage (e.g., local-lvm for VMs, local for ISOs/templates).
Any additional storage added (e.g., NFS, ZFS, CEPH).

Network Bridge Configuration:
How you set up vmbr0, vmbr1, vmbr2, and vmbr3 in Proxmox.
Crucial for Single NIC Isolation with Internet Access:
vmbr0 will be shared for both Proxmox management and pfSense WAN.
Example of /etc/network/interfaces for single NIC isolated lab with internet access:
# /etc/network/interfaces

auto lo
iface lo inet loopback

# Physical network adapter for Proxmox Management and Lab Internet/WAN access
# (e.g., eno1 - connect this to your home network/router)
auto eno1
iface eno1 inet manual

# Shared Bridge for Proxmox Management AND Lab Internet/WAN (pfSense WAN will connect here)
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24  # Replace with your actual Proxmox management IP in your home network
    gateway 192.168.1.1     # Replace with your actual home router's IP
    bridge-ports eno1       # Bridge to your physical NIC connected to home network
    bridge-stp off
    bridge-fd 0

# LAN Bridge (Internal - Management Network for your VMs - Kali Linux)
auto vmbr1
iface vmbr1 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0

# DMZ Bridge (Vulnerable Zone - Metasploitable 2/3, Windows Client)
auto vmbr2
iface vmbr2 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0

# Blue Team Tools Network Bridge (Wazuh, TheHive)
auto vmbr3
iface vmbr3 inet manual
    bridge-ports none
    bridge-stp off
    bridge-fd 0


Warning: After changing /etc/network/interfaces, you will need to restart networking (systemctl restart networking), reboot your Proxmox server or click on the "Apply Configuration" button. Ensure you have console access in case of issues. Your Proxmox web interface will only be accessible via the vmbr0 IP on your home network.
