 __  __      _                  _       _ _        _     _      _____ 
|  \/  | ___| |_ __ _ ___ _ __ | | ___ (_) |_ __ _| |__ | | ___|___ / 
| |\/| |/ _ \ __/ _` / __| '_ \| |/ _ \| | __/ _` | '_ \| |/ _ \ |_ \ 
| |  | |  __/ || (_| \__ \ |_) | | (_) | | || (_| | |_) | |  __/___) |
|_|  |_|\___|\__\__,_|___/ .__/|_|\___/|_|\__\__,_|_.__/|_|\___|____/ 
                         |_|                                          

Metasploitable3 is a more modern, Windows-based vulnerable machine. It's built using Packer and Vagrant. Getting it into Proxmox requires some conversion.

1. *Specifications:*
    - CPU: 2-4 Cores (Windows Server can be resource-intensive).
    - RAM: 4 GB (minimum), 8 GB (recommended).
    - Disk Size: 65 GB (minimum, as per official requirements), 80-100 GB recommended.
    - Network Interface (NIC): 1. Connected to your internal lab network (e.g. Vulnerable DMZ). Use VirtIO.
    - OS Type: Microsoft Windows -> Windows Server 2008 R2 (or similar, depending on the base image).
    - SCSI Controller: VirtIO SCSI.
    - QEMU Guest Agent: Enabled.

2. *How to Create on Proxmox:*
    Metasploitable3 is distributed as Vagrant boxes, which are typically VirtualBox or VMware images. You'll need to convert the .vmdk or .qcow2 from the Vagrant box.

    - *Prepare Metasploitable3:*
        - Install Vagrant and VirtualBox/VMware Workstation on a separate machine (or your Proxmox host if you really want, but not recommended for production).
        - Follow the Metasploitable3 instructions on their GitHub to build the VM (this will create the .vmdk or .qcow2 file): [https://github.com/rapid7/metasploitable3]

    - *Example for Windows:*
        - mkdir metasploitable3-workspace
        - cd metasploitable3-workspace
        - Invoke-WebRequest -Uri "https://raw.githubusercontent.com/rapid7/metasploitable3/master/Vagrantfile" -OutFile "Vagrantfile"
        - vagrant up win2k8 # This will download and build the Windows 2008 R2 image

        - Once vagrant up completes, find the generated .vmdk file. It's usually in a hidden Vagrant directory (e.g., ~/.vagrant.d/boxes/rapid7-VAGRANTSLASH-metasploitable3-win2k8/0.1.0/virtualbox/)
        - Upload VMDK to Proxmox Host: Transfer the .vmdk file to your Proxmox host (e.g., /var/lib/vz/template/iso/).


3. Create a New VM (without a disk initially): (Similar to Metasploitable2, but adjust specifications)

    - Click Create VM.
        - General:
            - Name: Metasploitable3.
        - OS:
            -Do not use any media.
            - Guest OS: Microsoft Windows -> Windows 7/2008r2 (or similar).
        - System:
            - SCSI Controller: VirtIO SCSI.
            - QEMU Guest Agent: Enable.
        - Disks: 
            - Delete the default disk.
        - CPU:
            - Cores: 2-4.
            - Type: host.
        - Memory:
            - Memory (MiB): 4096 (4GB) or 8192 (8GB).
        - Network:
            - Bridge: Vulnerable DMZ
            - Model: VirtIO (paravirtualized).
            - Firewall: Unchecked.
        - Confirm: Review and click Finish. Note the VM ID (e.g., 102).

4. *Import the VMDK Disk:*
    - Open the Proxmox Shell.
    - Use qm importdisk:
    - qm importdisk <VM_ID> <path_to_vmdk_file> <storage_name>
    # Example: qm importdisk 102 /var/lib/vz/template/iso/metasploitable3-win2k8-disk001.vmdk local-lvm

5. Attach the Imported Disk to the VM: (Similar to Metasploitable2).
    - Select Unused Disk 0, Edit, Bus/Device: SCSI (SCSI0).
    - Set Boot Order: Ensure the attached disk is first.
    - Start VM: Start the Metasploitable3 VM.
    - Default Credentials: vagrant/vagrant (may change after first boot/provisioning).
    - Troubleshooting: If you encounter BSODs, try changing the disk bus from SCSI to IDE in the VM hardware settings.