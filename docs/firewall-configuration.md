Basic Configuration in pfSense:
*NOTE*
THIS IS ONLY AN EXAPMLE, THE RULES YOU ADD TO THE FIREWALL WILL DEPEND ON WHAT YOU WANT TO HAVE

********************************************************************************************************************************************************************************************
Configuration Needed (MOST CRITICAL FOR SINGLE NIC):
Identify your Home Network Subnet(s): E.g., 192.168.1.0/24.
Identify your Proxmox Host's IP: E.g., 192.168.1.10.
pfSense WAN Rules (Firewall -> Rules -> WAN tab):
Add Block rules (move to top) for traffic coming from your home network subnet(s) (192.168.1.0/24) destined for your internal lab networks (192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24).
Example rule: Action: Block, Interface: WAN, Protocol: Any, Source: Network (Your Home Network Subnet e.g. 192.168.1.0/24), Destination: Any. (Adjust as needed if your home network needs any access to pfSense's services, but for strict isolation, block it all).
pfSense Internal Interface Rules (Firewall -> Rules -> MANAGEMENTATTACKERLAN, VULNERABLEZONEDMZ, BLUETEAMTOOLSNETWORK tabs):
Add Block rules (move to top) for traffic originating from these interfaces destined for your home network subnet(s) or the Proxmox host's IP.
Example rule for LAN: Action: Block, Interface: MANAGEMENTATTACKERLAN, Protocol: Any, Source: MANAGEMENTATTACKERLAN subnets, Destination: Network (Your Home Network Subnet e.g. 192.168.1.0/24).
Example rule for LAN (blocking Proxmox host): Action: Block, Interface: MANAGEMENTATTACKERLAN, Protocol: Any, Source: MANAGEMENTATTACKERLAN subnets, Destination: Host (Your Proxmox Host IP e.g., 192.168.1.10).
Repeat similar block rules on the VULNERABLEZONEDMZ and BLUETEAMTOOLSNETWORK interfaces.
Order of Rules: Ensure these Block rules are placed above any "Allow to WAN" rules on the respective interfaces. pfSense processes rules from top to bottom.
Finally, ensure you have "Allow" rules for outbound traffic to WAN net or any that allow internet access on your internal interfaces. These Pass to Internet rules must be below all specific Block rules and specific Pass rules.
********************************************************************************************************************************************************************************************


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

firewall interfaces [assets/Pfsenseinterfaces.png]