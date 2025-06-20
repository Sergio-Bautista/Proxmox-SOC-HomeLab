Kali Linux (Virtual Machine)
Your primary attacking machine.

Specifications:
CPU: 2-4 Cores (more if running multiple tools concurrently).
RAM: 2 GB (minimum for GUI), 4-8 GB (recommended for smooth operation and heavier tools like Burp Suite, Metasploit).
Disk Size: 30 GB (minimum for full install), 50-80 GB (recommended for tools, updates, and data).
Network Interface (NIC): 1. Connected to your internal lab network (e.g., vmbr1). Use VirtIO.
OS Type: Linux -> Debian.
SCSI Controller: VirtIO SCSI.
QEMU Guest Agent: Enabled.


How to Create on Proxmox:
Download Kali Linux ISO: Get the latest installer ISO (e.g., kali-linux-*-installer-amd64.iso) from the official Kali Linux website. [https://www.kali.org/get-kali/#kali-platforms]
Upload ISO to Proxmox: (Similar to other ISO uploads).


Create New VM:
Click Create VM.
General:
Name: Kali-Attacker.

OS: ISO image: Select your Kali Linux ISO.
Guest OS: Linux -> Debian.
System: Graphic card: Default (VGA).
SCSI Controller: VirtIO SCSI.
QEMU Guest Agent: Enable.

Disks:
Bus/Device: SCSI (SCSI0).
Storage: Select your VM storage.
Disk size (GiB): 50 (or more).
Cache: Write back.

CPU: Cores: 2-4.
Type: host.

Memory: Memory (MiB): 4096 (4GB) or 8192 (8GB).

Network: Bridge: vmbr1.
Model: VirtIO (paravirtualized).
Firewall: Unchecked.
Confirm: Review and click Finish.
Set Boot Order: Ensure the CD/DVD drive is first.

Start VM and Install Kali Linux:
Start the Kali-Attacker VM.
Open Console.
Follow the Kali Linux installation wizard. During disk partitioning, choose "Guided - Use entire disk".
when the intallation is done ensure you get an ip address from your selected network (e.g 10.0.0.0)