 _  __     _ _   _     _                  
| |/ /__ _| (_) | |   (_)_ __  _   ___  __
| ' // _` | | | | |   | | '_ \| | | \ \/ /
| . \ (_| | | | | |___| | | | | |_| |>  < 
|_|\_\__,_|_|_| |_____|_|_| |_|\__,_/_/\_\

1. *Specifications:*
    - CPU: 2-4 Cores (more if running multiple tools concurrently).
    - RAM: 2 GB (minimum for GUI), 4-8 GB (recommended for smooth operation and heavier tools like Burp Suite, Metasploit).
    - Disk Size: 30 GB (minimum for full install), 50-80 GB (recommended for tools, updates, and data).
    - Network Interface (NIC): Connected to your internal lab network (e.g.Management/Attacker). Use VirtIO.
    - OS Type: Linux -> Debian.
    - SCSI Controller: VirtIO SCSI.
    - QEMU Guest Agent: Enabled.


2. *How to Create on Proxmox:*
    - Download Kali Linux ISO: Get the latest installer ISO (e.g., kali-linux-*-installer-amd64.iso) from the official Kali Linux website. [https://www.kali.org/get-kali/#kali-platforms]
    - Upload ISO to Proxmox: (Similar to other ISO uploads).


3. *Create New VM:*
    - Click Create VM.

    - General:
        - Name: Kali-Attacker (Or chose your own name)
    - OS: 
        - ISO image: Select your Kali Linux ISO.
        - Guest OS: Linux -> Debian.
    - System: 
        - Graphic card: Default (VGA).
        - SCSI Controller: VirtIO SCSI.
        - QEMU Guest Agent: Enable.
    - Disks:
        - Bus/Device: SCSI (SCSI0).
        - Storage: Select your VM storage.
        - Disk size (GiB): 50 (or more).
        - Cache: Write back.
    - CPU: Cores: 2-4.
        - Type: host.
    - Memory: 
        - Memory (MiB): 4096 (4GB) or 8192 (8GB).
    - Network: 
        - Bridge: (Management/Attacker LAN)
        - Model: VirtIO (paravirtualized).
        - Firewall: checked.
    - Confirm: Review and click Finish.
    - Set Boot Order: Ensure the CD/DVD drive is first.


- Start VM and Install Kali Linux:
- Start the Kali-Attacker VM.
- Open Console.
- Follow the Kali Linux installation wizard. During disk partitioning, choose "Guided - Use entire disk".
- when the intallation is done ensure you get an ip address from your selected network (e.g 10.0.0.0)