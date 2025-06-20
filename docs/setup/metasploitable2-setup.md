 __  __      _                  _       _ _        _     _      ____   
|  \/  | ___| |_ __ _ ___ _ __ | | ___ (_) |_ __ _| |__ | | ___|___ \  
| |\/| |/ _ \ __/ _` / __| '_ \| |/ _ \| | __/ _` | '_ \| |/ _ \ __) | 
| |  | |  __/ || (_| \__ \ |_) | | (_) | | || (_| | |_) | |  __// __/  
|_|  |_|\___|\__\__,_|___/ .__/|_|\___/|_|\__\__,_|_.__/|_|\___|_____| 
                         |_|                                           

Metasploitable2 is a deliberately vulnerable Linux VM. It's quite old but still useful for basic penetration testing practice.

1. *Specifications:*
    - CPU: 1 Core.
    - RAM: 512 MB - 1 GB.
    - Disk Size: 8-10 GB.
    - Network Interface (NIC): 1. Connected to your internal lab network (e.g.Vulnerable DMZ). Use VirtIO.
    - OS Type: Linux -> Debian.
    - SCSI Controller: VirtIO SCSI.
    - QEMU Guest Agent: Not strictly necessary, but can be helpful.

2. *How to Create on Proxmox:*
    -  Metasploitable2 is typically distributed as a VM image (e.g., VMDK). You'll need to convert and import it.
    - Download Metasploitable2 VMDK: 
        - Get it from SourceForge: [https://sourceforge.net/projects/metasploitable/files/Metasploitable2/] (download the .zip which contains the .vmdk).
    - Extract the VMDK: Unzip the downloaded file.
    - Upload VMDK to Proxmox Host (via SCP/SFTP/SSH):
    - Use a tool like Filezilla (Windows) or scp (Linux/macOS) or use SSH to transfer the .vmdk file to your Proxmox host, e.g., to /var/lib/vz/template/iso/ or a temporary directory.
    - Create a New VM (without a disk initially):

3. *Click Create VM:*
    - General:
        - Name: Metasploitable2 (Or chose your own)
    - OS:
        - Do not use any media.
    - Guest OS: 
        - Linux -> Debian.
    - System:
        - SCSI Controller: 
            - VirtIO SCSI.
    - QEMU Guest Agent: 
        - (Optional, enable).
    - Disks: 
        - Delete the default disk (click "Delete" next to "scsi0").
    - CPU:
        - Cores: 1.
        - Type: host.
    - Memory:
        - Memory (MiB): 1024 (1GB).
    - Network:
        - Bridge: 
            - Vulnerable DMZ
        - Model: 
            - VirtIO (paravirtualized).
        - Firewall: 
        -   Unchecked.
    - Confirm: Review and click Finish. Note the VM ID (e.g., 101).

4. Import the VMDK Disk:
    - Open the Proxmox Shell (either via Web GUI or SSH).
    - Navigate to the directory where you uploaded the .vmdk file. (e.g., to /var/lib/vz/template/iso/)
    - Use qm importdisk to import the disk into your newly created VM:
    - qm importdisk <VM_ID> <path_to_vmdk_file> <storage_name>
        # Example: qm importdisk 101 /var/lib/vz/template/iso/Metasploitable-Linux-2.0.0.vmdk local-lvm
        - Replace <VM_ID> with your VM's ID (e.g., 101), <path_to_vmdk_file> with the actual path, and <storage_name> with your Proxmox storage where you want the VM disk to reside (e.g., local-lvm).
    

5. *Attach the Imported Disk to the VM:*
    - In the Proxmox Web GUI, select your Metasploitable2 VM.
    - Go to Hardware.
    - You will see an "Unused Disk 0". Select it and click Edit.
    - Bus/Device: Select SCSI (SCSI0).
    - Click Add.
    - Set Boot Order:
    - Go to Options -> Boot Order.
    - Ensure the newly attached disk (e.g., scsi0) is first.
    - Start VM: Start the Metasploitable2 VM. It should boot up.
    - Default Credentials: msfadmin/msfadmin