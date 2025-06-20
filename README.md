# Proxmox-SOC-HomeLab
····················································································
: __  __         _   _                        ____   ___   ____   _          _     :
:|  \/  |_   _  | | | | ___  _ __ ___   ___  / ___| / _ \ / ___| | |    __ _| |__  :
:| |\/| | | | | | |_| |/ _ \| '_ ` _ \ / _ \ \___ \| | | | |     | |   / _` | '_ \ :
:| |  | | |_| | |  _  | (_) | | | | | |  __/  ___) | |_| | |___  | |__| (_| | |_) |:
:|_|  |_|\__, | |_| |_|\___/|_| |_| |_|\___| |____/ \___/ \____| |_____\__,_|_.__/ :
:        |___/                                                                     :
····················································································

This repository documents the setup and configuration of my home Security Operations Center (SOC) lab, 
designed for practicing offensive and defensive cybersecurity skills in a controlled environment.

················································································
:  _____     _     _               __    ____            _             _       :
: |_   _|_ _| |__ | | ___    ___  / _|  / ___|___  _ __ | |_ ___ _ __ | |_ ___ :
:   | |/ _` | '_ \| |/ _ \  / _ \| |_  | |   / _ \| '_ \| __/ _ \ '_ \| __/ __|:
:   | | (_| | |_) | |  __/ | (_) |  _| | |__| (_) | | | | ||  __/ | | | |_\__ \:
:   |_|\__,_|_.__/|_|\___|  \___/|_|    \____\___/|_| |_|\__\___|_| |_|\__|___/:
:                                                                              :
················································································
1.-Lab Overview
2.-Hardware & Software Stack
3.-Network Topology
4.-Network Connectivity and Firewall Rules Explained
5.-Setup Guides
    Proxmox VE Server Setup
    pfSense Firewall Setup
    Wazuh SIEM/XDR Setup
    Kali Linux VM Setup
    Windows 10/Server Client VM Setup
    Metasploitable 2 (Linux) VM Setup
    Metasploitable 3 (Windows) VM Setup
6.-Post-Setup & Initial Configuration
7.-Usage & Learning Objectives
F8.-uture Plans
9.-Troubleshooting Notes
10Credits & Resources
  _          _        ___                       _               
 | |    __ _| |__    / _ \__   _____ _ ____   _(_) _____      __
 | |   / _` | '_ \  | | | \ \ / / _ \ '__\ \ / / |/ _ \ \ /\ / /
 | |__| (_| | |_) | | |_| |\ V /  __/ |   \ V /| |  __/\ V  V / 
 |_____\__,_|_.__/   \___/  \_/ \___|_|    \_/ |_|\___| \_/\_/                                             
    This lab provides a simulated network environment to:
    Practice penetration testing techniques using Kali Linux.
    Monitor network and endpoint activities with Wazuh.
    Analyze security incidents and respond to threats.
    Understand firewall rules and network segmentation with pfSense.
    Safely experiment with intentionally vulnerable systems.
  _   _               _                           ___     ____         __ _                            ____  _             _    
 | | | | __ _ _ __ __| |_      ____ _ _ __ ___   ( _ )   / ___|  ___  / _| |___      ____ _ _ __ ___  / ___|| |_ __ _  ___| | __
 | |_| |/ _` | '__/ _` \ \ /\ / / _` | '__/ _ \  / _ \/\ \___ \ / _ \| |_| __\ \ /\ / / _` | '__/ _ \ \___ \| __/ _` |/ __| |/ /
 |  _  | (_| | | | (_| |\ V  V / (_| | | |  __/ | (_>  <  ___) | (_) |  _| |_ \ V  V / (_| | | |  __/  ___) | || (_| | (__|   < 
 |_| |_|\__,_|_|  \__,_| \_/\_/ \__,_|_|  \___|  \___/\/ |____/ \___/|_|  \__| \_/\_/ \__,_|_|  \___| |____/ \__\__,_|\___|_|\_\
    Host Machine:
        Hardware: [Old Custom PC, CPU: Ryzen 7 1700x, RAM 32gb @ 3200Mhz, Storage: 256gb SSD (OS), Two 2TB Hard Drives (RAID 1 - mirror) for VM disks. Requires only one physical network interface card (NIC)]
        Operating System: Proxmox VE [Version 8.x]

    Virtual Machines:
        pfSense: VMs Firewall/Router
        Wazuh: SIEM/XDR (Manager and Dashboard)
        TheHive: Free Security Incident Response Platform
        Kali Linux: Attacker Workstation
        Windows Client: Standard user workstation (Windows 10)
        Metasploitable 2: Vulnerable Linux Target
        Metasploitable 3: Vulnerable Windows Server 2008 R2 Target
        Windows Server: For Active Directory (Future)
        Windows/Linux Machines: Add more targets (Future)
  _   _      _                      _      _____                 _                   
 | \ | | ___| |___      _____  _ __| | __ |_   _|__  _ __   ___ | | ___   __ _ _   _ 
 |  \| |/ _ \ __\ \ /\ / / _ \| '__| |/ /   | |/ _ \| '_ \ / _ \| |/ _ \ / _` | | | |
 | |\  |  __/ |_ \ V  V / (_) | |  |   <    | | (_) | |_) | (_) | | (_) | (_| | |_| |
 |_| \_|\___|\__| \_/\_/ \___/|_|  |_|\_\   |_|\___/| .__/ \___/|_|\___/ \__, |\__, |
                                                    |_|                  |___/ |___/ 
Example Description:
    The lab uses several virtual networks (Proxmox Bridges) managed by pfSense:
        Shared WAN/Proxmox Management: vmbr0 (bridged to physical NIC eno1). This bridge handles both Proxmox host management from your home network and provides internet access to the lab via pfSense's WAN interface.
    Lab Management/Attacker Network (LAN): 
        vmbr1 (10.0.0.0/24) - Where Kali Linux will reside.
    Vulnerable Zone (DMZ): 
        vmbr2 (10.0.10.0/24) - Where Metasploitable 2/3 and your Windows Client VM will reside.
    Blue Team Tools Network: 
        vmbr3 (10.0.20.0/24) - Where Wazuh and TheHive will reside.

[assets/NetworkDiagram.png]

  _   _      _                      _       ____                            _   _       _ _                           _  
 | \ | | ___| |___      _____  _ __| | __  / ___|___  _ __  _ __   ___  ___| |_(_)_   _(_) |_ _   _    __ _ _ __   __| | 
 |  \| |/ _ \ __\ \ /\ / / _ \| '__| |/ / | |   / _ \| '_ \| '_ \ / _ \/ __| __| \ \ / / | __| | | |  / _` | '_ \ / _` | 
 | |\  |  __/ |_ \ V  V / (_) | |  |   <  | |__| (_) | | | | | | |  __/ (__| |_| |\ V /| | |_| |_| | | (_| | | | | (_| | 
 |_|_\_|\___|\__| \_/\_/ \___/|_|_ |_|\_\__\____\___/|_| |_|_| |_|\___|\___|\__|_|_\_/ |_|\__|\__, |  \__,_|_| |_|\__,_| 
 |  ___(_)_ __ _____      ____ _| | | |  _ \ _   _| | ___  ___  | ____|_  ___ __ | | __ _(_)_ |___/___  __| |            
 | |_  | | '__/ _ \ \ /\ / / _` | | | | |_) | | | | |/ _ \/ __| |  _| \ \/ / '_ \| |/ _` | | '_ \ / _ \/ _` |            
 |  _| | | | |  __/\ V  V / (_| | | | |  _ <| |_| | |  __/\__ \ | |___ >  <| -|_) | | (_| | | | | |  __/ (_| |            
 |_|   |_|_|  \___| \_/\_/ \__,_|_|_| |_| \_\\__,_|_|\___||___/ |_____/_/\_\ .__/|_|\__,_|_|_| |_|\___|\__,_|            
                                                                           |_|
With pfSense as your central firewall, it acts as the gatekeeper between all your virtual networks. 
This means default connectivity between different segments is generally blocked by the firewall until you explicitly allow it
:  ____               __ _                        _ _                        __ _                       _   _                             _  :
: / ___|  ___  ___   / _(_)_ __ _____      ____ _| | |       ___ ___  _ __  / _(_) __ _ _   _ _ __ __ _| |_(_) ___  _ __    _ __ ___   __| | :
: \___ \ / _ \/ _ \ | |_| | '__/ _ \ \ /\ / / _` | | |_____ / __/ _ \| '_ \| |_| |/ _` | | | | '__/ _` | __| |/ _ \| '_ \  | '_ ` _ \ / _` | :
:  ___) |  __/  __/ |  _| | | |  __/\ V  V / (_| | | |_____| (_| (_) | | | |  _| | (_| | |_| | | | (_| | |_| | (_) | | | |_| | | | | | (_| | :
: |____/ \___|\___| |_| |_|_|  \___| \_/\_/ \__,_|_|_|    _ \___\___/|_| |_|_| |_|\__, |\__,_|_|  \__,_|\__|_|\___/|_| |_(_)_| |_| |_|\__,_| :
:  / _| ___  _ __   _ __ ___   ___  _ __ ___    __| | ___| |_ __ _(_) |___        |___/                                                      
: | |_ / _ \| '__| | '_ ` _ \ / _ \| '__/ _ \  / _` |/ _ \ __/ _` | | / __|                                                                  
: |  _| (_) | |    | | | | | | (_) | | |  __/ | (_| |  __/ || (_| | | \__ \                                                                  
: |_|  \___/|_|    |_| |_| |_|\___/|_|  \___|  \__,_|\___|\__\__,_|_|_|___/                                                  


  ____       _                  ____       _     _           
 / ___|  ___| |_ _   _ _ __    / ___|_   _(_) __| | ___  ___ 
 \___ \ / _ \ __| | | | '_ \  | |  _| | | | |/ _` |/ _ \/ __|
  ___) |  __/ |_| |_| | |_) | | |_| | |_| | | (_| |  __/\__ \
 |____/ \___|\__|\__,_| .__/   \____|\__,_|_|\__,_|\___||___/
                      |_|                                    
See [setup] for more details


  ____           _       ____       _                  ___     ___       _ _   _       _    ____             __ _                       _   _             
 |  _ \ ___  ___| |_    / ___|  ___| |_ _   _ _ __    ( _ )   |_ _|_ __ (_) |_(_) __ _| |  / ___|___  _ __  / _(_) __ _ _   _ _ __ __ _| |_(_) ___  _ __  
 | |_) / _ \/ __| __|___\___ \ / _ \ __| | | | '_ \   / _ \/\  | || '_ \| | __| |/ _` | | | |   / _ \| '_ \| |_| |/ _` | | | | '__/ _` | __| |/ _ \| '_ \ 
 |  __/ (_) \__ \ ||_____|__) |  __/ |_| |_| | |_) | | (_>  <  | || | | | | |_| | (_| | | | |__| (_) | | | |  _| | (_| | |_| | | | (_| | |_| | (_) | | | |
 |_|   \___/|___/\__|   |____/ \___|\__|\__,_| .__/   \___/\/ |___|_| |_|_|\__|_|\__,_|_|  \____\___/|_| |_|_| |_|\__, |\__,_|_|  \__,_|\__|_|\___/|_| |_|
                                             |_|                                                                  |___/                                   
Connectivity Tests:
    Verify all VMs can ping each other across the correct network segments. Verify VMs can reach the internet. Crucially, test that VMs CANNOT ping your Proxmox host's management IP or devices on your home network.
Wazuh Agent Enrollment: 
    Confirm all endpoints (Kali, Windows Client, Metasploitables) are reporting to the Wazuh Manager.
Initial Scans: 
    Run a basic Nmap scan from Kali against Metasploitable targets to confirm network reachability and open ports.
  _   _                         ___     _                          _                ___  _     _           _   _                
 | | | |___  __ _  __ _  ___   ( _ )   | |    ___  __ _ _ __ _ __ (_)_ __   __ _   / _ \| |__ (_) ___  ___| |_(_)_   _____  ___ 
 | | | / __|/ _` |/ _` |/ _ \  / _ \/\ | |   / _ \/ _` | '__| '_ \| | '_ \ / _` | | | | | '_ \| |/ _ \/ __| __| \ \ / / _ \/ __|
 | |_| \__ \ (_| | (_| |  __/ | (_>  < | |__|  __/ (_| | |  | | | | | | | | (_| | | |_| | |_) | |  __/ (__| |_| |\ V /  __/\__ \
  \___/|___/\__,_|\__, |\___|  \___/\/ |_____\___|\__,_|_|  |_| |_|_|_| |_|\__, |  \___/|_.__// |\___|\___|\__|_| \_/ \___||___/
                  |___/                                                    |___/            |__/
Red Teaming: 
    Practice vulnerability scanning, exploitation, post-exploitation.
Blue Teaming: 
    Monitor alerts in Wazuh, investigate logs, perform incident response.
Purple Teaming: 
    Combine red and blue team exercises to improve detection and response capabilities.
  _____      _                    ____  _                 
 |  ___|   _| |_ _   _ _ __ ___  |  _ \| | __ _ _ __  ___ 
 | |_ | | | | __| | | | '__/ _ \ | |_) | |/ _` | '_ \/ __|
 |  _|| |_| | |_| |_| | | |  __/ |  __/| | (_| | | | \__ \
 |_|   \__,_|\__|\__,_|_|  \___| |_|   |_|\__,_|_| |_|___/
    Add an Active Directory Domain Controller.
    Integrate a threat intelligence feed into Wazuh.
    Explore SOAR (Security Orchestration, Automation, and Response) tools.
    Add more intentionally vulnerable applications (e.g., OWASP Juice Shop).
  _____                _     _           _                 _   _               _   _       _            
 |_   _| __ ___  _   _| |__ | | ___  ___| |__   ___   ___ | |_(_)_ __   __ _  | \ | | ___ | |_ ___  ___ 
   | || '__/ _ \| | | | '_ \| |/ _ \/ __| '_ \ / _ \ / _ \| __| | '_ \ / _` | |  \| |/ _ \| __/ _ \/ __|
   | || | | (_) | |_| | |_) | |  __/\__ \ | | | (_) | (_) | |_| | | | | (_| | | |\  | (_) | ||  __/\__ \
   |_||_|  \___/ \__,_|_.__/|_|\___||___/_| |_|\___/ \___/ \__|_|_| |_|\__, | |_| \_|\___/ \__\___||___/
                                                                       |___/
"System Recovery Options" loop on Metasploitable 3: 
    Solved by changing VM disk Bus/Device from SCSI to IDE in Proxmox.

Ctrl+Alt+Delete issue: 
    Solved by using the Proxmox console's dedicated button or Ctrl+Alt+Insert.

Network connectivity issues: 
    Verify Proxmox bridges, pfSense firewall rules (especially the new isolation rules), and VM network adapter types (E1000 vs. VirtIO).

Wazuh Agent not connecting: 
    Check firewall rules, agent configuration (manager IP), and manager logs.
   ____              _ _ _          ___     ____                                         
  / ___|_ __ ___  __| (_) |_ ___   ( _ )   |  _ \ ___  ___  ___  _   _ _ __ ___ ___  ___ 
 | |   | '__/ _ \/ _` | | __/ __|  / _ \/\ | |_) / _ \/ __|/ _ \| | | | '__/ __/ _ \/ __|
 | |___| | |  __/ (_| | | |_\__ \ | (_>  < |  _ <  __/\__ \ (_) | |_| | | | (_|  __/\__ \
  \____|_|  \___|\__,_|_|\__|___/  \___/\/ |_| \_\___||___/\___/ \__,_|_|  \___\___||___/
    Proxmox VE Official Website - [https://www.proxmox.com/en/]
    pfSense Official Website - [https://www.pfsense.org/]
    Wazuh Official Documentation - [https://documentation.wazuh.com/current/index.html]
    Kali Linux Official Website - [https://www.kali.org/]
    Metasploitable 2 - [https://docs.rapid7.com/metasploit/metasploitable-2/]
    Metasploitable 3 GitHub - [https://github.com/rapid7/metasploitable3]
    Fedora VirtIO Drivers (for virtio-win.iso) [https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/]
    Check out this tutorial - [https://www.youtube.com/watch?v=VuSKMPRXN1M&t=764s]

