Connectivity Tests:
Verify all VMs can ping each other across the correct network segments. 
Verify VMs can reach the internet. 
Crucially, test that VMs CANNOT ping your Proxmox host's management IP or devices on your home network.

Wazuh Agent Enrollment: 
Confirm all endpoints (Kali, Windows Client, Metasploitables) are reporting to the Wazuh Manager.