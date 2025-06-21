````
__        ___           _                     _  ___                     __ _                       _   _              
\ \      / (_)_ __   __| | _____      _____  / |/ _ \    ___ ___  _ __  / _(_) __ _ _   _ _ __ __ _| |_(_) ___  _ __   
 \ \ /\ / /| | '_ \ / _` |/ _ \ \ /\ / / __| | | | | |  / __/ _ \| '_ \| |_| |/ _` | | | | '__/ _` | __| |/ _ \| '_ \  
  \ V  V / | | | | | (_| | (_) \ V  V /\__ \ | | |_| | | (_| (_) | | | |  _| | (_| | |_| | | | (_| | |_| | (_) | | | | 
   \_/\_/  |_|_| |_|\__,_|\___/ \_/\_/ |___/ |_|\___/   \___\___/|_| |_|_| |_|\__, |\__,_|_|  \__,_|\__|_|\___/|_| |_| 
                                                                              |___/                                    
````
1. *Installing Sysmon*

- Once the windows VM has been installed, we need to donwload sysmon to help us monitor and log system activity.
     - Download the sysmon from the offical microsoft page [here](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
     (NOTE if you are using linux donload it from the github respoitory)
     - Download the configuration file we need to configure sysmon, you download it [here](https://github.com/olafhartong/sysmon-modular)
        - Scroll down until you see the **sysmonconfig.xml** file 
        - click **raw** button on the top right of the page (or click the download raw file)
        - right click and "save as" as dave it on your computer as anything you want (Reemember the name)
        - Open up a powershell command prompt with admin privilages
        - Navegate to the folder where you extracted the sysmon files (Make sure that the sysmonconfig.xml file is also on this folder)
        - Run the command to install sysmon .\Sysmon64.exe -i <name_of_your_sysmonconfig.xml_file> (e.g .\Sysmon64.exe -i sysmonconfig.xml)
        - Accept the Licence & Agreement, and sysmon should install 

2. *Check if sysmon installed correctly*
    - Go to Windows --> type Services --> and look for the sysmon service running
    - Windows --> Event Viewer --> Applications & Services Logs --> Microsoft --> Windows --> sysmon should be running