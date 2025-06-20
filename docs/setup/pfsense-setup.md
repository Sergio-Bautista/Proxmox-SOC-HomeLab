pfSense Firewall Setup

Specifications for pfSense VM:

CPU: 2 Cores (more for heavy throughput or complex rule sets).
RAM: 2 GB (minimum), 4 GB (recommended for features like packages, VPN, logging).
Disk Size: 16-32 GB (sufficient for logs and packages).
Network Interfaces (NICs): 4. All using VirtIO (paravirtualized) for best performance.
OS Type: Other -> FreeBSD.
SCSI Controller: VirtIO SCSI (recommended for disk performance).


Download pfSense ISO

Go to the official pfSense website [https://www.pfsense.org/download/].
Select:
Architecture: AMD64 (64-bit)
Installer: USB Memstick Installer (or DVD image if you prefer)
Platform: USB Memstick Installer (or the appropriate one for your download method)
Click Download.


Upload pfSense ISO to Proxmox

Log in to your Proxmox Web GUI.
In the left-hand navigation pane, select your Proxmox node (e.g., pve).
Under your node, click on your ISO storage (e.g., local or a dedicated ISO storage).
In the main content area, click on ISO Images.
Click the Upload button.
Click Select File, browse to your downloaded pfSense ISO, and click Upload. Wait for "TASK OK".


Create Linux Bridges on Proxmox

For each of your four interfaces on pfSense, you'll need a corresponding Linux Bridge on Proxmox.

In the Proxmox Web GUI, go to Datacenter -> your_node -> Network.

You will likely already have vmbr0 (your Proxmox management bridge, often connected to a physical NIC and providing internet access to your Proxmox host).

Create additional bridges: Click Create -> Linux Bridge. Repeat this for three more bridges (you'll use vmbr0 for WAN).

vmbr1 (Management/Attacker):
Name: vmbr1
IPv4/CIDR: None (pfSense will handle DHCP and IP addressing for this network).
Bridge ports: Leave blank (unless you want to bind it to a specific physical NIC for a dedicated LAN, which is rare in a lab setting where it's internal).
Click Create.

vmbr2 (OPT1 / DMZ):
Name: vmbr2
IPv4/CIDR: None
Bridge ports: Leave blank.
Click Create.

vmbr3 (OPT2 / Blue team tools):
Name: vmbr3
IPv4/CIDR: None
Bridge ports: Leave blank.
Click Create.

After creating them, click Apply Configuration at the top right of the Network interface page to make the changes active. A reboot of the Proxmox host is sometimes necessary for complex network changes, but usually not for just adding new internal bridges.


Boot up the pfsense VM and follow the installations instructions, make sure you use the correct interfaces for each WAN/LAN

Interface Assignment (Crucial Step):
After reboot, pfSense will present the Interface Assignment menu in the console.
It will list the detected network interfaces (e.g., vtnet0, vtnet1, vtnet2, vtnet3) with their MAC addresses.
Should VLANs be set up now? Type n and press Enter.
Enter the WAN interface name: Identify the MAC address that corresponds to vmbr0 in Proxmox (the first NIC you added). Type its vtnetX name (e.g., vtnet0) and press Enter.
Enter the LAN interface name: Identify the MAC address for vmbr1. Type its vtnetX name (e.g., vtnet1) and press Enter.
Enter the Optional 1 interface name: Identify the MAC address for vmbr2. Type its vtnetX name (e.g., vtnet2) and press Enter.
Enter the Optional 2 interface name: Identify the MAC address for vmbr3. Type its vtnetX name (e.g., vtnet3) and press Enter.
Do you want to proceed? Type y and press Enter.
pfSense will apply the interface assignments and then present the main console menu.























Network Adapters:
WAN: net0 - Intel E1000, connected to vmbr0 (the shared bridge)
LAN: net1 - Intel E1000, connected to vmbr1 (Lab Management/Attacker Network)
DMZ: net2 - Intel E1000, connected to vmbr2 (Vulnerable Zone)
Blue Team Net: net3 - Intel E1000, connected to vmbr3 (Blue Team Tools Network)

Disk: IDE (e.g., 8GB)

pfSense Installation Steps:

Mount ISO to VM CD-ROM.

Initial console setup (assigning interfaces WAN/LAN/DMZ/Blue Team Net etc.).

Setting WAN IP address to DHCP (it will get an IP from your home router via vmbr0) or use a Static ip if preferred.

Setting initial IP addresses for LAN, DMZ, Blue Team Net interfaces (e.g., 192.168.10.1/24, 192.168.20.1/24, 192.168.30.1/24).
Enabling SSH (optional, for management).
Accessing the pfSense web GUI (from a VM on vmbr1 like Kali, or via a temporary physical connection).








Basic Configuration in pfSense:
Setting up DHCP servers for LAN, DMZ, Blue Team Net networks.

CRITICAL FIREWALL RULES FOR ISOLATION:
Outbound NAT: Ensure default outbound NAT rules are enabled on the WAN.

WAN Interface Rules (Firewall -> Rules -> WAN tab):
Add Block rules (move to top) for traffic coming from your home network subnet(s) (192.168.1.0/24) destined for your internal lab networks (192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24).
Example rule: Action: Block, Interface: WAN, Protocol: Any, Source: Network (Your Home Network Subnet e.g. 192.168.1.0/24), Destination: Any. (Adjust as needed if your home network needs any access to pfSense's services, but for strict isolation, block it all).

Internal Interface Rules (Firewall -> Rules -> MANAGEMENTATTACKERLAN, VULNERABLEZONEDMZ, BLUETEAMTOOLSNETWORK tabs):
Add Block rules (move to top) for traffic originating from these interfaces destined for your home network subnet(s) or the Proxmox host's IP.
Example rule for LAN: Action: Block, Interface: MANAGEMENTATTACKERLAN, Protocol: Any, Source: MANAGEMENTATTACKERLAN subnets, Destination: Network (Your Home Network Subnet e.g. 192.168.1.0/24).
Example rule for LAN (blocking Proxmox host): Action: Block, Interface: MANAGEMENTATTACKERLAN, Protocol: Any, Source: MANAGEMENTATTACKERLAN subnets, Destination: Host (Your Proxmox Host IP e.g., 192.168.1.10).

Repeat similar block rules on the VULNERABLEZONEDMZ and BLUETEAMTOOLSNETWORK interfaces.

Crucial Inter-VLAN Blocking: Add explicit Block rules for traffic between your internal lab networks to enforce segmentation. Place these blocks above any broad Pass rules on each interface.
On MANAGEMENTATTACKERLAN (Kali's Network - 10.0.0.0/24) interface:

Rule 1: BLOCK LAN to Blue Team Net (General Block)
Action: Block, Interface: MANAGEMENTATTACKERLAN, Protocol: Any, Source: MANAGEMENTATTACKERLAN subnets, Destination: BLUETEAMTOOLSNETWORK subnets. Description: BLOCK LAN to Blue Team Net
Purpose: Prevents Kali from arbitrarily connecting to Wazuh/TheHive, increasing Blue Team isolation.

Rule 2: BLOCK LAN to DMZ (General Block)
Action: Block, Interface: MANAGEMENTATTACKERLAN, Protocol: Any, Source: MANAGEMENTATTACKERLAN subnets, Destination: VULNERABLEZONEDMZ subnets. Description: BLOCK LAN to DMZ
Purpose: Acts as a default deny to all DMZ traffic from Kali, forcing you to define specific attack/testing protocols.

Rule 3: ALLOW Kali to DMZ (Attacks - Specific Protocols)
Action: Pass, Interface: MANAGEMENTATTACKERLAN, Protocol: Any (or ICMP for ping, TCP/UDP for specific attacks like scanning/exploitation), Source: MANAGEMENTATTACKERLAN subnets, Destination: 
VULNERABLEZONEDMZ subnets. Description: ALLOW Kali to DMZ (Attacks)
Pingability to DMZ from Kali: YES (if ICMP is included).
Position: Place above the BLOCK LAN to DMZ (Rule 2). If you prefer more granular control for attacks, remove the broad BLOCK LAN to DMZ and add only specific Pass rules for ports/protocols you intend to attack (e.g., TCP/80, TCP/443, TCP/22, TCP/445).

Rule 4: ALLOW LAN to Blue Team Net (Wazuh/TheHive Access - Specific Protocols)
Action: Pass, Interface: MANAGEMENTATTACKERLAN, Protocol: TCP (and UDP if needed), Source: MANAGEMENTATTACKERLAN subnets, Destination: Host (IP of Wazuh/TheHive machine, e.g., 10.0.20.100), Destination Port Range: 80, 443 (for web UIs), 22 (for SSH), 8080, 5601 (if Kibana/Wazuh Dashboards are on non-standard ports). Description: ALLOW Kali to Wazuh/TheHive Management
Pingability to Blue Team Net from Kali: YES (if ICMP is included in "Any" or specifically allowed).
Position: Place above the BLOCK LAN to Blue Team Net (Rule 1).


On VULNERABLEZONEDMZ (Targets Network - 10.0.10.0/24) interface:
Rule 1: BLOCK DMZ to Management/Attacker LAN
Action: Block, Interface: VULNERABLEZONEDMZ, Protocol: Any, Source: VULNERABLEZONEDMZ subnets, Destination: MANAGEMENTATTACKERLAN subnets. Description: BLOCK DMZ to Management/Attacker LAN
Purpose: Prevents compromised DMZ machines from attacking Kali directly.
Pingability to Management/Attacker LAN from DMZ: NO.

Rule 2: BLOCK DMZ to Blue Team Tools Network (General Block)
Action: Block, Interface: VULNERABLEZONEDMZ, Protocol: Any, Source: VULNERABLEZONEDMZ subnets, Destination: BLUETEAMTOOLSNETWORK subnets. Description: BLOCK DMZ to Blue Team Tools Net
Purpose: Default deny for DMZ to Blue Team, allowing only specific monitoring traffic.
Pingability to Blue Team Net from DMZ: NO (unless specifically allowed by Rule 3).

Rule 3: ALLOW DMZ Agents to Wazuh Manager (for monitoring)
Action: Pass, Interface: VULNERABLEZONEDMZ, Protocol: TCP, Source: VULNERABLEZONEDMZ subnets (or specific IPs of monitored machines, e.g., 10.0.10.10 for Windows), Destination: Host (IP of Wazuh Manager, e.g., 10.0.20.100), Destination Port Range: 1514 (Wazuh agent communication), 55000 (Wazuh agent active response/control). Description: ALLOW DMZ Agents to Wazuh Manager
Pingability for Wazuh Agent traffic from DMZ to Wazuh Manager: NO (unless ICMP is explicitly allowed alongside these TCP ports). Wazuh agents primarily use TCP for communication, so ICMP ping is not strictly necessary for agent functionality.
Position: Place above the BLOCK DMZ to Blue Team Tools Network (Rule 2).


On BLUETEAMTOOLSNETWORK (Wazuh/TheHive Network - 10.0.20.0/24) interface:
Rule 1: BLOCK Blue Team to DMZ (General Block)
Action: Block, Interface: BLUETEAMTOOLSNETWORK, Protocol: Any, Source: BLUETEAMTOOLSNETWORK subnets, Destination: VULNERABLEZONEDMZ subnets. Description: BLOCK Blue Team to DMZ
Purpose: Default deny for Blue Team to DMZ, allowing only specific management traffic.
Pingability to DMZ from Blue Team Net: NO (unless specifically allowed by Rule 3).

Rule 2: BLOCK Blue Team to Management/Attacker LAN (General Block)
Action: Block, Interface: BLUETEAMTOOLSNETWORK, Protocol: Any, Source: BLUETEAMTOOLSNETWORK subnets, Destination: MANAGEMENTATTACKERLAN subnets. Description: BLOCK Blue Team to Management/Attacker LAN
Purpose: Default deny for Blue Team to Management/Attacker LAN.
Pingability to Management/Attacker LAN from Blue Team Net: NO (unless specifically allowed by Rule 4).

Rule 3: ALLOW Blue Team to DMZ (for agent checks/TheHive actions - specific protocols)
Action: Pass, Interface: BLUETEAMTOOLSNETWORK, Protocol: Any (or specific protocols needed by Wazuh Manager/TheHive to interact with DMZ machines, e.g., SSH, WinRM, ICMP for pingability testing). Source: BLUETEAMTOOLSNETWORK subnets, Destination: VULNERABLEZONEDMZ subnets. Description: ALLOW Blue Team to DMZ (management/actions)
Pingability to DMZ from Blue Team Net: YES (if ICMP is explicitly added or "Any" is used). This is useful for testing connectivity from your blue team tools.
Position: Place above the BLOCK Blue Team to DMZ (Rule 1).

Rule 4: ALLOW Blue Team to Management/Attacker LAN (for management/updates - specific protocols)
Action: Pass, Interface: BLUETEAMTOOLSNETWORK, Protocol: Any (or specific protocols like SSH, HTTP/S for management). Source: BLUETEAMTOOLSNETWORK subnets, Destination: MANAGEMENTATTACKERLAN subnets. Description: ALLOW Blue Team to Management/Attacker LAN (for management/updates)
Pingability to Management/Attacker LAN from Blue Team Net: YES (if ICMP is explicitly added or "Any" is used).
Position: Place above the BLOCK Blue Team to Management/Attacker LAN (Rule 2).
Finally, ensure you have "Allow" rules for outbound traffic to WAN net or any that allow internet access on your internal interfaces. These Pass to Internet rules must be below all specific Block rules and specific Pass rules.
