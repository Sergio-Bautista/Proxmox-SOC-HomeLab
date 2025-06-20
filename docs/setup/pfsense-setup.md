````
        __ ____                        _____ _                        _ _   ____       _               
 _ __  / _/ ___|  ___ _ __  ___  ___  |  ___(_)_ __ _____      ____ _| | | / ___|  ___| |_ _   _ _ __  
| '_ \| |_\___ \ / _ \ '_ \/ __|/ _ \ | |_  | | '__/ _ \ \ /\ / / _` | | | \___ \ / _ \ __| | | | '_ \ 
| |_) |  _|___) |  __/ | | \__ \  __/ |  _| | | | |  __/\ V  V / (_| | | |  ___) |  __/ |_| |_| | |_) |
| .__/|_| |____/ \___|_| |_|___/\___| |_|   |_|_|  \___| \_/\_/ \__,_|_|_| |____/ \___|\__|\__,_| .__/ 
|_|                                                                                             |_|    
````
1. *Specifications for pfSense VM:*
    - CPU: 2 Cores (more for heavy throughput or complex rule sets).
    - RAM: 2 GB (minimum), 4 GB (recommended for features like packages, VPN, logging).
    - Disk Size: 16-32 GB (sufficient for logs and packages).
    - Network Interfaces (NICs): 4. All using VirtIO (paravirtualized) for best performance.
    - OS Type: Other -> FreeBSD.
    - SCSI Controller: VirtIO SCSI (recommended for disk performance).
    - Disk: IDE (e.g., 8GB)


2. *Download pfSense ISO*
    -Go to the official pfSense website [https://www.pfsense.org/download/].
    -Select:
        - Architecture: AMD64 (64-bit)
        - Installer: USB Memstick Installer (or DVD image if you prefer)
        - Platform: USB Memstick Installer (or the appropriate one for your download method)
        - Click Download.


3. *Upload pfSense ISO to Proxmox*
    - Log in to your Proxmox Web GUI.
    - In the left-hand navigation pane, select your Proxmox node (e.g., pve).
    - Under your node, click on your ISO storage (e.g., local or a dedicated ISO storage).
    - In the main content area, click on ISO Images.
    - Click the Upload button.
    - Click Select File, browse to your downloaded pfSense ISO, and click Upload. Wait for "TASK OK".


4. *Create Linux Bridges on Proxmox*
    - For each of your four interfaces on pfSense, you'll need a corresponding Linux Bridge on Proxmox.
    - In the Proxmox Web GUI, go to Datacenter -> your_node -> Network.
    - You will likely already have vmbr0 (your Proxmox management bridge, often connected to a physical NIC and providing internet access to your Proxmox host).
    
    - Create additional bridges: 
        - Click Create -> Linux Bridge. Repeat this for three more bridges (you'll use vmbr0 for WAN).
        - vmbr1 (Management/Attacker):
        - Name: vmbr1
        - IPv4/CIDR: None (pfSense will handle DHCP and IP addressing for this network).
        - Bridge ports: Leave blank (unless you want to bind it to a specific physical NIC for a dedicated LAN, which is rare in a lab setting where it's internal).
    - Click Create.

        - vmbr2 (OPT1 / DMZ):
        - Name: vmbr2
        - IPv4/CIDR: None
        - Bridge ports: Leave blank.
    - Click Create.

        - vmbr3 (OPT2 / Blue team tools):
        - Name: vmbr3
        - IPv4/CIDR: None
        - Bridge ports: Leave blank.
    - Click Create.

- After creating them, click Apply Configuration at the top right of the Network interface page to make the changes active. A reboot of the Proxmox host is sometimes necessary for complex network changes, but usually not for just adding new internal bridges.


- Boot up the pfsense VM and follow the installations instructions, make sure you use the correct interfaces for each WAN/LAN

5. *Interface Assignment (Crucial Step):*
    - After reboot, pfSense will present the Interface Assignment menu in the console.
    - It will list the detected network interfaces (e.g., vtnet0, vtnet1, vtnet2, vtnet3) with their MAC addresses.
    - Should VLANs be set up now? Type n and press Enter.
    - Enter the WAN interface name: Identify the MAC address that corresponds to vmbr0 in Proxmox (the first NIC you added). Type its vtnetX name (e.g., vtnet0) and press Enter.
    - Enter the LAN interface name: Identify the MAC address for vmbr1. Type its vtnetX name (e.g., vtnet1) and press Enter.
    - Enter the Optional 1 interface name: Identify the MAC address for vmbr2. Type its vtnetX name (e.g., vtnet2) and press Enter.
    - Enter the Optional 2 interface name: Identify the MAC address for vmbr3. Type its vtnetX name (e.g., vtnet3) and press Enter.
    - Do you want to proceed? Type y and press Enter.
    - pfSense will apply the interface assignments and then present the main console menu.

6. *Network Adapters:*
    - WAN: net0 - Intel E1000, connected to vmbr0 (the shared bridge WAN)
    - LAN: net1 - Intel E1000, connected to vmbr1 (Lab Management/Attacker Network)
    - DMZ: net2 - Intel E1000, connected to vmbr2 (Vulnerable Zone)
    - Blue Team Net: net3 - Intel E1000, connected to vmbr3 (Blue Team Tools Network)

7. *pfSense Installation Steps:*
    - Mount ISO to VM CD-ROM.
    - Initial console setup (assigning interfaces WAN/LAN/DMZ/Blue Team Net etc.).
    - Setting WAN IP address to DHCP (it will get an IP from your home router via vmbr0) or use a Static ip if preferred.
    - Setting initial IP addresses for LAN, DMZ, Blue Team Net interfaces (e.g., 192.168.10.1/24, 192.168.20.1/24, 192.168.30.1/24).
    - Enabling SSH (optional, for management).
    - Accessing the pfSense web GUI (from a VM on vmbr1 like Kali, or via a temporary physical connection).

8. *Basic Configuration in pfSense:*

    *CRITICAL FIREWALL RULES FOR ISOLATION:*
    
    - Outbound NAT: Ensure default outbound NAT rules are enabled on the WAN.
    1. WAN Interface Rules (Firewall -> Rules -> WAN tab):
        - Add Block rules (move to top) for traffic coming from your home network subnet(s) (192.168.1.0/24) destined for your internal lab networks (192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24).
        - Example rule: 
            - Action: Block
            - Interface: WAN
            - Protocol: Any
            - Source: Network (Your Home Network Subnet e.g. 192.168.1.0/24)
            - Destination: Any (Adjust as needed if your home network needs any access to pfSense's services, but for strict isolation, block it all).

    2. Internal Interface Rules (Firewall -> Rules -> MANAGEMENTATTACKERLAN, - VULNERABLEZONEDMZ, BLUETEAMTOOLSNETWORK tabs):    
        - Add Block rules (move to top) for traffic originating from these interfaces destined for your home network subnet(s) or the Proxmox host's IP.
        - Example rule for LAN: 
            - Action: Block
            - Interface: MANAGEMENTATTACKERLAN
            - Protocol: Any
            - Source: MANAGEMENTATTACKERLAN subnets 
            - Destination: Network (Your Home Network Subnet e.g. 192.168.1.0/24).
        - Example rule for LAN (blocking Proxmox host): 
        - Action: Block
        - Interface: MANAGEMENTATTACKERLAN
        - Protocol: Any
        - Source: MANAGEMENTATTACKERLAN subnets
        - Destination: Host (Your Proxmox Host IP e.g., 192.168.1.10).

*Repeat similar block rules on the VULNERABLEZONEDMZ and BLUETEAMTOOLSNETWORK interfaces.*

See [firewall-configuration.md](../firewall-configuration.md) for Inter VLAN rules 