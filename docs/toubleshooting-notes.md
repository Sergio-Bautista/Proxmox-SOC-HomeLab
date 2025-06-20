"System Recovery Options" loop on Metasploitable 3: 
Solved by changing VM disk Bus/Device from SCSI to IDE in Proxmox.

Ctrl+Alt+Delete issue: 
Solved by using the Proxmox console's dedicated button or Ctrl+Alt+Insert.

Network connectivity issues:
Verify Proxmox bridges, pfSense firewall rules (especially the new isolation rules), and VM network adapter types (E1000 vs. VirtIO).

Wazuh Agent not connecting:
Check firewall rules, agent configuration (manager IP), and manager logs