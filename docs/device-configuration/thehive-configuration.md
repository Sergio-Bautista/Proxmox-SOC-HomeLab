````
 _____ _          _   _ _              ____             __ _                       _   _             
|_   _| |__   ___| | | (_)_   _____   / ___|___  _ __  / _(_) __ _ _   _ _ __ __ _| |_(_) ___  _ __  
  | | | '_ \ / _ \ |_| | \ \ / / _ \ | |   / _ \| '_ \| |_| |/ _` | | | | '__/ _` | __| |/ _ \| '_ \ 
  | | | | | |  __/  _  | |\ V /  __/ | |__| (_) | | | |  _| | (_| | |_| | | | (_| | |_| | (_) | | | |
  |_| |_| |_|\___|_| |_|_| \_/ \___|  \____\___/|_| |_|_| |_|\__, |\__,_|_|  \__,_|\__|_|\___/|_| |_|
                                                             |___/                                   
````

We have to edit cassandra's files for the to be able to communicate with Wazuh 

1. *Configure Cassandra Database*
  - Open the cassandra configuration file using nano /etc/cassandra/cassandra.ymal
  - Change the following lines
    - Cluster_name: "Soc-Lab" #This could be whatever name you like [check](../assets/cassandra-cluster_name.png)
    - Listen_address: <IP_of_TheHive> (e.g, 10.0.20.101) [check](../assets/cassandra-Listen_address.png)
    - rpc_address: <IP_of_TheHive> [check](../assets/cassandra-rpc_address.png)
    - Under "seed_provider" we need to change:
      -seeds: <IP_of_TheHive>:7000 (e.g, 10.0.20.101:7000) [check](../assets/cassandra_seed.png)
    - Save this file and exit 
    - Stop cassandra service 
      - systemctl stop cassandra.service  
    - Remove old files
      - rm -rf /var/lib/cassandra/*
    - Once the old files are deleted, restart the cassandra service 
      - systemctl start cassandra.service
    - Type systemctl status cassandra.service
    **Cassandra.service shoudl be running**
  


2. *Configure elasticsearch file*: Used to manage and query data
  - Open the elasticsearch file with nano /etc/elasticsearch/elasticsearch.yml
  - Scroll down and uncomment "Cluster_name" and chage it to "TheHive" [check](../assets/elasticsearch-Cluster_name.png)
  - leave "Node.Name", "path.data", and "path.logs" as is [check](../assets/elasticsearch-node_namepng.png)
  - Scroll down until you see "network.host" and change to <IP_of_TheHive> [check](../assets/elasticsearch-netwokr_host.png)
  - In the "Discovery" section, uncomment either "cluster.initial_master_nodes" or "discovery.seed_hosts"
    - This is required in order to start elasticsearch. the discovery seed is used to scale out elasticsearch
  - type: systemctl start elasticsearch
  - then type: systemctl enable elasticsearch
  - check the elasticsearch status (systemctl status elasticsearch)
  **elasticsearch.service should be running**

**CRITICAL STEP**
 - type: chown -R thehive:thehive /opt/thp 
 this will ensure that thehive has access to the correct files it needs


3. Edit TheHive configuration file
  - Open the file with: nano /etc/thehive/configuration.conf 
  - Scroll down until you see "hostname" and change it to <IP_of_TheHive> also change the "cluster-name" you chose earlier [check this](../assets/thehive-config.png)
  - The storage section piont to a directory where the thehive stores the files this is why we change the owner earlier so the hive could have access to this files [check](../assets/thehive-storage-url.png)
  - Scroll down until you see "application/baseURL" and change it to <IP_of_TheHive> [check](../assets/thehive-storage-url.png)
  - Enable thethive
    - systemctl start thehive
    - systemctl enable thehive
    - at this point make sure that all three services are running, (cassandra, elasticsearch, and thehive)
    - to access the dashboard check out [thehive-setup.md](../setup/thehive-setup.md)