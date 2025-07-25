# Proxmox-SOC-HomeLab
````
····················································································
: __  __         _   _                        ____   ___   ____   _          _     :
:|  \/  |_   _  | | | | ___  _ __ ___   ___  / ___| / _ \ / ___| | |    __ _| |__  :
:| |\/| | | | | | |_| |/ _ \| '_ ` _ \ / _ \ \___ \| | | | |     | |   / _` | '_ \ :
:| |  | | |_| | |  _  | (_) | | | | | |  __/  ___) | |_| | |___  | |__| (_| | |_) |:
:|_|  |_|\__, | |_| |_|\___/|_| |_| |_|\___| |____/ \___/ \____| |_____\__,_|_.__/ :
:        |___/                                                                     :
····················································································
````
This repository documents the setup and configuration of my home Security Operations Center (SOC) lab, 
designed for practicing offensive and defensive cybersecurity skills in a controlled environment.


````
················································································
:  _____     _     _               __    ____            _             _       :
: |_   _|_ _| |__ | | ___    ___  / _|  / ___|___  _ __ | |_ ___ _ __ | |_ ___ :
:   | |/ _` | '_ \| |/ _ \  / _ \| |_  | |   / _ \| '_ \| __/ _ \ '_ \| __/ __|:
:   | | (_| | |_) | |  __/ | (_) |  _| | |__| (_) | | | | ||  __/ | | | |_\__ \:
:   |_|\__,_|_.__/|_|\___|  \___/|_|    \____\___/|_| |_|\__\___|_| |_|\__|___/:
:                                                                              :
················································································

````
1. Lab Overview
2. Hardware & Software Stack
3. Network Topology
4. Network Connectivity and Firewall Rules Explained
5. Setup Guides    
    - Proxmox VE Server Setup
    - pfSense Firewall Setup
    - Wazuh SIEM/XDR Setup
    - Kali Linux VM Setup
    - Windows 10/Server Client VM Setup
    - Metasploitable 2 (Linux) VM Setup
    - Metasploitable 3 (Windows) VM Setup
6. Post-Setup & Initial Configuration
7. Usage & Learning Objectives
8. uture Plans
9. Troubleshooting Notes
10. Credits & Resources

````
  _          _        ___                       _               
 | |    __ _| |__    / _ \__   _____ _ ____   _(_) _____      __
 | |   / _` | '_ \  | | | \ \ / / _ \ '__\ \ / / |/ _ \ \ /\ / /
 | |__| (_| | |_) | | |_| |\ V /  __/ |   \ V /| |  __/\ V  V / 
 |_____\__,_|_.__/   \___/  \_/ \___|_|    \_/ |_|\___| \_/\_/                                             
                                          
````

*This lab provides a simulated network environment to:*

Practice penetration testing techniques using Kali Linux.
Monitor network and endpoint activities with Wazuh.
Analyze security incidents and respond to threats.
Understand firewall rules and network segmentation with pfSense.
Safely experiment with intentionally vulnerable systems.

````
  _   _    _    ____  ______        ___    ____  _____    ___     ____   ___  _____ _______        ___    ____  _____   ____ _____  _    ____ _  __ 
 | | | |  / \  |  _ \|  _ \ \      / / \  |  _ \| ____|  ( _ )   / ___| / _ \|  ___|_   _\ \      / / \  |  _ \| ____| / ___|_   _|/ \  / ___| |/ / 
 | |_| | / _ \ | |_) | | | \ \ /\ / / _ \ | |_) |  _|    / _ \/\ \___ \| | | | |_    | |  \ \ /\ / / _ \ | |_) |  _|   \___ \ | | / _ \| |   | ' /  
 |  _  |/ ___ \|  _ <| |_| |\ V  V / ___ \|  _ <| |___  | (_>  <  ___) | |_| |  _|   | |   \ V  V / ___ \|  _ <| |___   ___) || |/ ___ \ |___| . \  
 |_| |_/_/   \_\_| \_\____/  \_/\_/_/   \_\_| \_\_____|  \___/\/ |____/ \___/|_|     |_|    \_/\_/_/   \_\_| \_\_____| |____/ |_/_/   \_\____|_|\_\ 
                                                                                                                                                    
````
*Host Machine:*

*Hardware:* Old Custom PC 
        
    - CPU: Ryzen 7 1700x 
    - RAM 32gb @ 3200Mhz (future upgrade)
    - Storage: 256gb SSD (OS)
        Two 2TB Hard Drives (RAID 1 - mirror) for VM disks.
    - Requires only one physical network interface card (NIC)
    - Operating System: Proxmox VE [Version 8.x]

*Virtual Machines:*

    1. pfSense: VMs Firewall/Router
    2. Wazuh: SIEM/XDR (Manager and Dashboard)
    3. TheHive: Free Security Incident Response Platform
    4. Kali Linux: Attacker Workstation    
    5. Windows Client: Standard user workstation (Windows 10)
    6. Metasploitable 2: Vulnerable Linux Target
    7. Metasploitable 3: Vulnerable Windows Server 2008 R2 Target
    8. Windows Server: For Active Directory (Future)
    9. Windows/Linux Machines: Add more targets (Future)

````
  _   _      _                      _      _____                 _                   
 | \ | | ___| |___      _____  _ __| | __ |_   _|__  _ __   ___ | | ___   __ _ _   _ 
 |  \| |/ _ \ __\ \ /\ / / _ \| '__| |/ /   | |/ _ \| '_ \ / _ \| |/ _ \ / _` | | | |
 | |\  |  __/ |_ \ V  V / (_) | |  |   <    | | (_) | |_) | (_) | | (_) | (_| | |_| |
 |_| \_|\___|\__| \_/\_/ \___/|_|  |_|\_\   |_|\___/| .__/ \___/|_|\___/ \__, |\__, |
                                                    |_|                  |___/ |___/ 

````
*Example Description:*

The lab uses several virtual networks (Proxmox Bridges) managed by pfSense:    
    Shared WAN/Proxmox Management: vmbr0 (bridged to physical NIC eno1). 
    This bridge handles both Proxmox host management from your home network and provides internet access to the lab via pfSense's 
        
*WAN interface.*

1. Lab Management/Attacker Network (LAN): 
    - vmbr1 (10.0.0.0/24) - Where Kali Linux will reside.

2. Vulnerable Zone (DMZ):  
    - vmbr2 (10.0.10.0/24) - Where Metasploitable 2/3 and your Windows Client VM will reside.
   
3. Blue Team Tools Network: 
    - vmbr3 (10.0.20.0/24) - Where Wazuh and TheHive will reside.

````
                            +-------------------------+
                            |                         |
                            |        Internet         |
                            |                         |
                            +-----------^-------------+
                                        |
                                        |
            +---------------------------^---------------------------+ 
            |                                                       |
            |                                                       | 
+-----------^-------------+                             +-----------^----------+             
|      Home Netwrok       |                             |      Proxmox WAN     |               
|   (e.g, 192.168.0.1/24) |                             |       Interface      |
|       (router)          |                             |        (vmbr0)       |
+-----------^-------------+                             +-----------^----------+  
            |                                                       |
            |                                                       |
            +---------------------------|---------------------------+                                
                                        |
                            +-----------^-------------+
                            |      Network Switch     |
                            |        (Amazon)         |
                            +-----------^-------------+
                                        |
                                        |
                              +---------^--------+ 
                              |                  | 
                              |   Proxmox Host   |
                              |                  |
                              +---------^--------+  
                                        |
                                        |
                      +-----------------^-----------------+
                      |               vmbr0               |
                      |    (Shared WAN / Proxmox Mgmt)    |
                      +-----------------^-----------------+
                                        |
                                        |
                        +---------------^-------------+
                        |                             |
                        |  pfSense VM (Firewall)      |
                        |  WAN(vmbr0): DHCP from Home |
                        |                             |
                        +---------------^-------------+
                                        |
                                        |
                +-----------------------+-----------------------+
                |                       |                       |
       vmbr1 (10.0.0.0/24)      vmbr2 (10.0.10.0/24)    vmbr3 (10.0.20.0/24)
       (MGMT/ATTACKER LAN)       (VULNERABLE DMZ)        (BLUE TEAM TOOLS)
                |                       |                       |
        +-------^-------+       +-------^---------+     +-------^-------+
        |  Kali Linux   |       | Metasploitable2 |     |     Wazuh     |
        | (10.0.0.10)   |       | (10.0.10.100)   |     | (10.0.20.100) |
        +---------------+       +-------^---------+     +-------^-------+
                                        |                       |
                                +-------^---------+     +-------^-------+
                                | Metasploitable3 |     |   TheHive     |
                                | (10.0.10.101)   |     | (10.0.20.101) |
                                +-------^---------+     +---------------+
                                        |
                                +-------^---------+
                                | Windows Client  |
                                | (10.0.10.102)   |
                                +-----------------+
````

````
 _   _      _                      _       ____                            _   _       _ _                           _  
 | \ | | ___| |___      _____  _ __| | __  / ___|___  _ __  _ __   ___  ___| |_(_)_   _(_) |_ _   _    __ _ _ __   __| | 
 |  \| |/ _ \ __\ \ /\ / / _ \| '__| |/ / | |   / _ \| '_ \| '_ \ / _ \/ __| __| \ \ / / | __| | | |  / _` | '_ \ / _` | 
 | |\  |  __/ |_ \ V  V / (_) | |  |   <  | |__| (_) | | | | | | |  __/ (__| |_| |\ V /| | |_| |_| | | (_| | | | | (_| | 
 |_|_\_|\___|\__| \_/\_/ \___/|_|  |_|\_\__\____\___/|_| |_|_| |_|\___|\___|\__|_|_\_/ |_|\__|\__, |  \__,_|_| |_|\__,_| 
 |  ___(_)_ __ _____      ____ _| | | |  _ \ _   _| | ___  ___  | ____|_  ___ __ | | __ _(_)_ |___/___  __| |            
 | |_  | | '__/ _ \ \ /\ / / _` | | | | |_) | | | | |/ _ \/ __| |  _| \ \/ / '_ \| |/ _` | | '_ \ / _ \/ _` |            
 |  _| | | | |  __/\ V  V / (_| | | | |  _ <| |_| | |  __/\__ \ | |___ >  <| -|_) | | (_| | | | | |  __/ (_| |            
 |_|   |_|_|  \___| \_/\_/ \__,_|_|_| |_| \_\\__,_|_|\___||___/ |_____/_/\_\ .__/|_|\__,_|_|_| |_|\___|\__,_|            
                                                                           |_|
````
With pfSense as your central firewall, it acts as the gatekeeper between all your virtual networks. 
This means default connectivity between different segments is generally blocked by the firewall until you explicitly allow it

See [firewall-configuration.md](docs/firewall-configuration.md) 

````
           _                             _     _      
  ___  ___| |_ _   _ _ __     __ _ _   _(_) __| | ___ 
 / __|/ _ \ __| | | | '_ \   / _` | | | | |/ _` |/ _ \
 \__ \  __/ |_| |_| | |_) | | (_| | |_| | | (_| |  __/
 |___/\___|\__|\__,_| .__/   \__, |\__,_|_|\__,_|\___|
                    |_|      |___/                    
````
(Detailed setup instructions are located in the docs folder of this repository.)

1. [Proxmox VE Server Setup](docs/setup/proxmox-setup.md)
2. [pfSense Firewall Setup](docs/setup/pfsense-setup.md)
3. [Wazuh SIEM/XDR Setup](docs/setup/wazuh-setup.md)
4. [TheHive setup](docs/setup/thehive-setup.md)
6. [Kali Linux VM Setup](docs/setup/kali-linux-setup.md)
7. [Windows 10 Client VM Setup](docs/setup/windows10-setup.md)
8. [Metasploitable 2 (Linux) VM Setup](docs/setup/metasploitable2-setup.md)
9. [Metasploitable 3 (Windows) VM Setup](docs/setup/metasploitable3-setup.md)


````
  ____           _       ____       _                  ___     ___       _ _   _       _    ____             __ _                       _   _             
 |  _ \ ___  ___| |_    / ___|  ___| |_ _   _ _ __    ( _ )   |_ _|_ __ (_) |_(_) __ _| |  / ___|___  _ __  / _(_) __ _ _   _ _ __ __ _| |_(_) ___  _ __  
 | |_) / _ \/ __| __|___\___ \ / _ \ __| | | | '_ \   / _ \/\  | || '_ \| | __| |/ _` | | | |   / _ \| '_ \| |_| |/ _` | | | | '__/ _` | __| |/ _ \| '_ \ 
 |  __/ (_) \__ \ ||_____|__) |  __/ |_| |_| | |_) | | (_>  <  | || | | | | |_| | (_| | | | |__| (_) | | | |  _| | (_| | |_| | | | (_| | |_| | (_) | | | |
 |_|   \___/|___/\__|   |____/ \___|\__|\__,_| .__/   \___/\/ |___|_| |_|_|\__|_|\__,_|_|  \____\___/|_| |_|_| |_|\__, |\__,_|_|  \__,_|\__|_|\___/|_| |_|
                                             |_|                                                                  |___/                                   

````
1. *Connectivity Tests:*
    
    - Verify all VMs can ping each other across the correct network segments. 
    - Verify VMs can reach the internet. 
    - Crucially, test that VMs CANNOT ping your Proxmox host's management IP or devices on your home network.

2. *Wazuh Agent Enrollment:*
    
    - Confirm all endpoints (Kali, Windows Client, Metasploitables) are reporting to the Wazuh Manager.

3. *Initial Scans:*
    
    - Run a basic Nmap scan from Kali against Metasploitable targets to confirm network reachability and open ports.


````
  _   _                         ___     _                          _                ___  _     _           _   _                
 | | | |___  __ _  __ _  ___   ( _ )   | |    ___  __ _ _ __ _ __ (_)_ __   __ _   / _ \| |__ (_) ___  ___| |_(_)_   _____  ___ 
 | | | / __|/ _` |/ _` |/ _ \  / _ \/\ | |   / _ \/ _` | '__| '_ \| | '_ \ / _` | | | | | '_ \| |/ _ \/ __| __| \ \ / / _ \/ __|
 | |_| \__ \ (_| | (_| |  __/ | (_>  < | |__|  __/ (_| | |  | | | | | | | | (_| | | |_| | |_) | |  __/ (__| |_| |\ V /  __/\__ \
  \___/|___/\__,_|\__, |\___|  \___/\/ |_____\___|\__,_|_|  |_| |_|_|_| |_|\__, |  \___/|_.__// |\___|\___|\__|_| \_/ \___||___/
                  |___/                                                    |___/            |__/

````
1. *Red Teaming:* 
   
    - Practice vulnerability scanning, exploitation, post-exploitation.

2. *Blue Teaming:* 

    - Monitor alerts in Wazuh, investigate logs, perform incident response.

3. *Purple Teaming:* 
   
    - Combine red and blue team exercises to improve detection and response capabilities.


````
  _____      _                    ____  _                 
 |  ___|   _| |_ _   _ _ __ ___  |  _ \| | __ _ _ __  ___ 
 | |_ | | | | __| | | | '__/ _ \ | |_) | |/ _` | '_ \/ __|
 |  _|| |_| | |_| |_| | | |  __/ |  __/| | (_| | | | \__ \
 |_|   \__,_|\__|\__,_|_|  \___| |_|   |_|\__,_|_| |_|___/

````
Integrate a threat intelligence feed into Wazuh.
Explore SOAR (Security Orchestration, Automation, and Response) tools.
Add more intentionally vulnerable applications (e.g., OWASP Juice Shop).
 
````
  _____                _     _           _                 _   _               _   _       _            
 |_   _| __ ___  _   _| |__ | | ___  ___| |__   ___   ___ | |_(_)_ __   __ _  | \ | | ___ | |_ ___  ___ 
   | || '__/ _ \| | | | '_ \| |/ _ \/ __| '_ \ / _ \ / _ \| __| | '_ \ / _` | |  \| |/ _ \| __/ _ \/ __|
   | || | | (_) | |_| | |_) | |  __/\__ \ | | | (_) | (_) | |_| | | | | (_| | | |\  | (_) | ||  __/\__ \
   |_||_|  \___/ \__,_|_|__/|_|\___||___/_| |_|\___/ \___/ \__|_|_| |_|\__, | |_| \_|\___/ \__\___||___/
                                                                       |___/

````
1. *"System Recovery Options" loop on Metasploitable 3:* 
    - Solved by changing VM disk Bus/Device from SCSI to IDE in Proxmox.

2. *Ctrl+Alt+Delete issue:* 
    - Solved by using the Proxmox console's dedicated button or Ctrl+Alt+Insert.

3. *Network connectivity issues:* 
    - Verify Proxmox bridges, pfSense firewall rules (especially the new isolation rules), and VM network adapter types (E1000 vs. VirtIO).

4. *Wazuh Agent not connecting:* 
    - Check firewall rules, agent configuration (manager IP), and manager logs.

````
   ____              _ _ _          ___     ____                                         
  / ___|_ __ ___  __| (_) |_ ___   ( _ )   |  _ \ ___  ___  ___  _   _ _ __ ___ ___  ___ 
 | |   | '__/ _ \/ _` | | __/ __|  / _ \/\ | |_) / _ \/ __|/ _ \| | | | '__/ __/ _ \/ __|
 | |___| | |  __/ (_| | | |_\__ \ | (_>  < |  _ <  __/\__ \ (_) | |_| | | | (_|  __/\__ \
  \____|_|  \___|\__,_|_|\__|___/  \___/\/ |_| \_\___||___/\___/ \__,_|_|  \___\___||___/

````
1. Proxmox VE Official Website - [https://www.proxmox.com/en/]
2. pfSense Official Website - [https://www.pfsense.org/]
3. Wazuh Official Documentation - [https://documentation.wazuh.com/current/index.html]
4. Kali Linux Official Website - [https://www.kali.org/]
5. Metasploitable 2 - [https://docs.rapid7.com/metasploit/metasploitable-2/]
6. Metasploitable 3 GitHub - [https://github.com/rapid7/metasploitable3]
7. Fedora VirtIO Drivers (for virtio-win.iso) [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/]
8. Check out this tutorial - [https://www.youtube.com/watch?v=VuSKMPRXN1M&t=764s]

