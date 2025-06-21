````
__        __              _        ____             __ _                       _   _             
\ \      / /_ _ _____   _| |__    / ___|___  _ __  / _(_) __ _ _   _ _ __ __ _| |_(_) ___  _ __  
 \ \ /\ / / _` |_  / | | | '_ \  | |   / _ \| '_ \| |_| |/ _` | | | | '__/ _` | __| |/ _ \| '_ \ 
  \ V  V / (_| |/ /| |_| | | | | | |__| (_) | | | |  _| | (_| | |_| | | | (_| | |_| | (_) | | | |
   \_/\_/ \__,_/___|\__,_|_| |_|  \____\___/|_| |_|_| |_|\__, |\__,_|_|  \__,_|\__|_|\___/|_| |_|
                                                         |___/                                   
````


*How to Add agents to Wazuh*

1. Deplying a new agent to wazuh 
    - login into the wazuh dahsboard with the credentials you were given during the installation 
        - alternatively you can check the credentials [here](./wazuh-credentials.md)
2. Click on "Add Agent"
    - It will display different agents to chose from (linux, macOS, Windows) select your desired agent
    - In the "Server Address" section add your <Wazuh_IP> (e.g, 10.0.20.100) 
    - On the "Optional Settings" you can leave it as is, but its recommended to add a d3escriptive name to each agent you add (e.g, Windows10)
    - In step 4, copy the command given and run it on your agent device
    - In step 5, run the command "NET START WazuhSvc" or the command generated for you (NOTE this command is for windows agents)
3. After running the commands, you should see the Wazuh service running on your machine
    - to check if the service is running you can go to:
        - Windows --> services --> look for Wazuh service

- After these steps your reload you Wazuh dashboard and you should see your new agent
**YOU MIGHT NEED TO WAIT FOR THE AGENT TO CONNECT**
