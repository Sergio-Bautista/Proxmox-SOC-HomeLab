````
 ____                  _  __ _           _   _                    __              _____ _            _   _ _             _    __  ______ 
/ ___| _ __   ___  ___(_)/ _(_) ___ __ _| |_(_) ___  _ __  ___   / _| ___  _ __  |_   _| |__   ___  | | | (_)_   _____  | |   \ \/ / ___|
\___ \| '_ \ / _ \/ __| | |_| |/ __/ _` | __| |/ _ \| '_ \/ __| | |_ / _ \| '__|   | | | '_ \ / _ \ | |_| | \ \ / / _ \ | |    \  / |    
 ___) | |_) |  __/ (__| |  _| | (_| (_| | |_| | (_) | | | \__ \ |  _| (_) | |      | | | | | |  __/ |  _  | |\ V /  __/ | |___ /  \ |___ 
|____/| .__/ \___|\___|_|_| |_|\___\__,_|\__|_|\___/|_| |_|___/ |_|  \___/|_|      |_| |_| |_|\___| |_| |_|_| \_/ \___| |_____/_/\_\____|
      |_|                                                                                                                                
````
1. *Hardware Preparation:*
    - CT ID: Choose a unique ID (e.g., 104).
    - Hostname: thehive-lxc (or similar).
    - CPU: 4 Cores (minimum, more if you expect heavy usage or many concurrent users/analyzers).
    - RAM: 8 GB (minimum, 12GB+ recommended for Elasticsearch performance).
    - Swap: 2 GB (helps with memory spikes).
    - Disk Size: 80 GB (minimum, Elasticsearch and logs can consume significant space). Use fast storage (SSD/NVMe).
    - Network Interface: Bridged to your internal lab network (e.g., vmbr1). Assign a static IP address.
    - Template Type: Ubuntu 22.04 LTS Standard or Debian 11/12 Standard.
    - Features: nesting=1 and fscaps=1.

2. *Download the LXC Template*

- *(If you already downloaded an Ubuntu 22.04 Standard template for Wazuh, you can skip this.)*

    - Log in to your Proxmox Web GUI.
    - Navigate to your Proxmox node (e.g., pve).
    - Under your node, click on your local storage (e.g., local or local-lvm).
    - In the main content area, click on CT Templates.
    - Click the Templates button at the top.
    - Select ubuntu-22.04-standard (or a recent Debian standard template).
    - Click Download.

3. *Create the LXC Container*

    - In the Proxmox Web GUI, click the Create CT button in the top right corner.
    
    - General Tab:
        - Node: Select your Proxmox node.
        - CT ID: Enter a unique ID (e.g., 104).
        - Hostname: thehive-lxc
        - Password: Set a strong password for the root user. Confirm it.
    - Click Next.

    - Template Tab:
        - Template: Select the Ubuntu 22.04 (or Debian) standard template you downloaded.
    - Click Next.

    - Disk Tab:
        - Disk size (GiB): Enter 80 (or more).
    - Storage: Select your preferred storage for LXC containers (e.g., local-lvm).
    - Click Next.

    - CPU Tab:
        - Cores: Enter 4.
    - Click Next.

    - Memory Tab:
        - Memory (MiB): Enter 8192 (for 8GB).
        - Swap (MiB): Enter 2048 (for 2GB).
    - Click Next.

    - Network Tab:
        - Bridge: Select vmbr1 (your internal lab network bridge).
        - IPv4/CIDR: Select "Static" and enter a static IP address for your The Hive LXC (e.g., 192.168.1.104/24). Ensure this IP is within your vmbr1 range and not conflicting.
        - Gateway (IPv4): Enter the IP address of your pfSense LAN interface (e.g., 192.168.1.1).
    - Click Next.

    - DNS Tab:
        - Use host settings: Or specify your pfSense IP (192.168.1.1) or public DNS servers.
    - Click Next.
    
    - Confirm Tab:
        - Review all settings. Make sure "Start after created" is unchecked for now.
    - Click Finish.



4. *Installing TheHive 5*

- *Dependences*
    - apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb-release

- *Install Java*
    - wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor  -o /usr/share/keyrings/corretto.gpg
    - echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" |  sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
    - sudo apt update
    - sudo apt install java-common java-11-amazon-corretto-jdk
    - echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment 
    - export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"

- *Install Cassandra*
    - wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg
    - echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
    - sudo apt update
    - sudo apt install cassandra

- *Install ElasticSearch*
    - wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
    - sudo apt-get install apt-transport-https
    - echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list
    - sudo apt update
    - sudo apt install elasticsearch

- ***OPTIONAL ELASTICSEARCH***
    - -Create a jvm.options file under /etc/elasticsearch/jvm.options.d and put the following configurations in that file.
    - -Dlog4j2.formatMsgNoLookups=true
    - -Xms2g
    - -Xmx2g

- *Install TheHive*
    - wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
    - echo 'deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main' | sudo tee -a /etc/apt/sources.list.d/strangebee.list
    - sudo apt-get update
    - sudo apt-get install -y thehive


5. *Access The Hive Dashboard*
    - Once all containers are running, open a web browser on a machine connected to your vmbr1 network (e.g., your Kali Linux VM or Windows 10 VM).
    - Navigate to https://<Your_TheHive_LXC_IP_Address>/thehive (e.g., https://192.168.1.104/thehive).
    - You might encounter a "Your connection is not private" or "Untrusted connection" warning due to the self-signed SSL certificates generated by the init.sh script. You can safely proceed past this     warning for a lab environment.
    - Default Credentials on port 9000
    - credentials are 'admin@thehive.local' with a password of 'secret'