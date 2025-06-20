
Wazuh (LXC Container)
Wazuh is an open-source security platform. Running it as an LXC is efficient for resource usage.

Specifications:

CPU: 4 Cores (recommended for the Manager, especially with agents).
RAM: 4 GB (minimum for the Manager), 8 GB+ recommended (Elasticsearch/OpenSearch can be memory-intensive).
Disk Size: 18 GB (minimum for core Wazuh), significantly more for logs (e.g., 64 GB+ depending on the number of agents and retention). Use a fast storage type (SSD).
Network Interface (NIC): 1. Connected to your Blue Team Tools interface (e.g 10.0.20.0/24). Bridged mode.
Template Type: Ubuntu 22.04 LTS or Debian 11/12.
Features: nesting=1 (required for some Wazuh components if running within the LXC, though usually not strictly for a basic manager), fscaps=1 (filesystem capabilities).


Download the LXC Template

Log in to your Proxmox Web GUI.
In the left-hand navigation pane, select your Proxmox node (e.g., pve).
Under your node, click on your local storage (e.g., local or local-lvm).
In the main content area, click on CT Templates.
Click the Templates button at the top.
A window will pop up showing available templates. Scroll down and select ubuntu-22.04-standard (or a recent Debian standard template).
Click Download. Wait for the download to complete (you'll see "TASK OK").


Create the LXC Container

In the Proxmox Web GUI, click the Create CT button in the top right corner.

General Tab:
Node: Select your Proxmox node.
CT ID: Enter a unique ID (e.g., 103).
Hostname: wazuh-lxc
Password: Set a strong password for the root user. Confirm it.
Click Next.

Template Tab:
Template: Select the Ubuntu 22.04 (or Debian) standard template you just downloaded.
Click Next.

Disk Tab:
Disk size (GiB): Enter 64 (or more, based on your needs).
Storage: Select your preferred storage for LXC containers (e.g., local-lvm).
Click Next.

CPU Tab:
Cores: Enter 4.
Click Next.
Memory Tab:
Memory (MiB): Enter 8192 (for 8GB).
Swap (MiB): Enter 2048 (for 2GB).
Click Next.

Network Tab:
Bridge: Select(your blue team internal network bridge).
IPv4/CIDR: LEAVE BLANK YOU WILL GET AN IP ADDRESS FROM YOUR BLUE TEAM TOOLS NETWORK (e.g 10.0.20.0/24)
Leave IPv6 as "Static" and unconfigured unless you're using IPv6.
Click Next.
DNS Tab:
Use host settings: This is usually fine, or you can specify your pfSense IP or public DNS servers (e.g., 8.8.8.8).
Click Next.

Confirm Tab:
Review all settings. Make sure "Start after created" is unchecked for now, as we need to configure features first.
Click Finish.

Download and install Wazuh

Install Wazuh 4.7
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

This will take a while to download, after the installation is done, you will be given a username and a password to login into the wazuh dashboard