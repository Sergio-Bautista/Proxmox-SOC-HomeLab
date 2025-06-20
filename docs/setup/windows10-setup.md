__        ___           _                     _  ___     ____     ___      _               _   __  __            _     _          __  
\ \      / (_)_ __   __| | _____      _____  / |/ _ \   / /\ \   / (_)_ __| |_ _   _  __ _| | |  \/  | __ _  ___| |__ (_)_ __   __\ \ 
 \ \ /\ / /| | '_ \ / _` |/ _ \ \ /\ / / __| | | | | | | |  \ \ / /| | '__| __| | | |/ _` | | | |\/| |/ _` |/ __| '_ \| | '_ \ / _ \ |
  \ V  V / | | | | | (_| | (_) \ V  V /\__ \ | | |_| | | |   \ V / | | |  | |_| |_| | (_| | | | |  | | (_| | (__| | | | | | | |  __/ |
   \_/\_/  |_|_| |_|\__,_|\___/ \_/\_/ |___/ |_|\___/  | |    \_/  |_|_|   \__|\__,_|\__,_|_| |_|  |_|\__,_|\___|_| |_|_|_| |_|\___| |
                                                        \_\                                                                       /_/ 

1. *Specifications: *
    - CPU: 2-4 Cores (depending on usage).
    - RAM: 4 GB (minimum), 8 GB (recommended).
    - Disk Size: 60-100 GB (minimum, Windows needs space).
    - Network Interface (NIC): 1. Connected to your internal lab network (e.g., vmbr1 from pfSense). Use VirtIO.
    - OS Type: Microsoft Windows -> Windows 10/2016.
    - SCSI Controller: VirtIO SCSI (recommended).
    - QEMU Guest Agent: Enabled.

2. *How to Create on Proxmox:*
    - Download Windows 10 ISO: Get the official ISO from Microsoft. [https://www.microsoft.com/en-us/software-download/windows10]
    - Upload ISO to Proxmox: (Similar to pfSense ISO upload).

    - Create New VM:
    
    - Click Create VM.
    - General:
        - Name: Win10-Lab.
    - OS:
        - ISO image: Select your Windows 10 ISO.
        - Guest OS: Microsoft Windows -> Windows 10/2016.
    - System:
        - Graphic card: Default (VGA).
        - SCSI Controller: VirtIO SCSI.
        - QEMU Guest Agent: Enable.
    - Disks:
        - Bus/Device: SCSI (SCSI0).
        - Storage: Select your VM storage.
        - Disk size (GiB): 80 (or more).
        - Cache: Write back.
    - CPU:
        - Cores: 2-4.
        - Type: host.
    - Memory:
        - Memory (MiB): 4096 (4GB) or 8192 (8GB).
    - Network:
        - Bridge: vmbr1 (your internal lab network).
        - Model: VirtIO (paravirtualized).
        - Firewall: checked.
    - Confirm: Review and click Finish.
    - Set Boot Order: Ensure the CD/DVD drive is first.

3. *windows 10 Installation*
    - Start VM and Install Windows 10:
    - Start the Win10-Lab VM.
    - Open Console.
    - During Windows installation, when it asks for drivers for the disk, you will need to load the VirtIO SCSI driver. You can download the virtio-win.iso [here](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/) or use the one typically available on your Proxmox server (/usr/share/pve-edk2-firmware/virtio-win.iso). Mount this ISO as a second CD/DVD drive in your VM's hardware settings.
    - Select load drivers --> D:\vioscsi\win10\amd64\vioscsi.inf (this will display the virtusal disk)
    - Load drivers -->  D:\vioscsi\win10\amd64\netkvm.inf (this is for the network)
    - Once Windows is installed, install the QEMU Guest Agent and VirtIO drivers from the virtio-win.iso.
