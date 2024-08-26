# Active-Directory-Lab
Active Directory Domain to simulate managing user accounts and monitoring activity within the domain. 

This purpose of this project was to expose myself to hands-on experience with IT administration and build a splunk instance to gather telemetry attached to the domain and target machine. I wanted to extend the project beyond the typical AD deployment and implement an attacking machine within the network to demonstrate how sysmon and splunk work together to gather and deliver security events I configure within an input.conf file. I will also demonstrate how this configuration can be used to conduct vulnerability assesments using AtomicRedTeam.

### Skills Learned

- Understanding of the workflow and data collection possibilities of a SIEM.
- Ability to deploy a local Active Directory domain.
- Configure and organize users, groups, and organizational units within Active Directory. 
- Adept navigation within the Splunk web interface.
- Identifying dfferent attacks via data provided in a SIEM.
- Conducting a brute force attack on Kali Linux using Crowbar.
- Use AtomicRedTeam to identify vulnerabilities and harden the current security configuration within the domain.

### Tools Used

- Sysmon
- Splunk
- SplunkUniversalForwarder
- Crowbar brute force tool
- AtomicRedTeam 

## Steps
This is a diagram I made that represents the NAT Network I configured within VirtualBox for this lab. Each machine is labled and has a brief list of its function below the name. NAT Network allowed me to have the lab within its own isolated environment seperated from my host network while still having access to the outside internet.

![Screenshot 2024-08-25 210132](https://github.com/user-attachments/assets/ab3c7cbf-f73c-425f-b69d-e40b33508e38)

First, I need to install and configure each virtual machine to their operational states. I did not document the most of this process as I already had cloned machines to use for this project. This process is fairly self explanitory. Each machine was also configured with a static IP address either in the GUI or command line depending on the machine. 

Evidence of static IP configuration in the Ubuntu server (Splunk Server):
![Screenshot 2024-08-25 232232](https://github.com/user-attachments/assets/4bb4d106-15bf-4e11-ab1b-a20d794644d5)
![Screenshot 2024-08-25 211329](https://github.com/user-attachments/assets/fc8c2ea2-15b6-48c6-94cd-4f4e97de521d)



Installing splunk on the Ubuntu live server: 
![Screenshot 2024-08-25 213039](https://github.com/user-attachments/assets/687ac429-ac1a-4d5c-87aa-80fad6437353)
I first installed the VirtualBox guest additions to the machine which will give me the ability to utilize a shared folder where I have the Splunk Enterprise iso located. I then made a new directory named "share" in the Ubuntu server where I would then mount the contents of the shared folder from my host machine to. I also added the current user to the vboxsf group to give myself the required permissions to access the shared folder. I was unfamiliar with the mounting process from the linux command line when I reached this point. After a bit of research, I was able to input the command "sudo mount -t vboxsf -o uid=1000,gid=1000 Active_Directory_Project share/" which then transfered the contents of the share folder to my Ubuntu server. From here I followed the online documentation for the install process once the file is obtained. 
Installing Splunk Universal Forwarder is also a fairly easy process with the provided documentation found online, only major input I had to make was the receiving indexer IP and port number: 192.168.10.10:9997

Installing Sysmon: 
Sysmon is used to collect log events from the machine and enables the use of a SIEM agent to analyze and identify important security events on a given machine. I downloaded Sysmon from https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon. Then, I installed a custom configuration called "Sysmon-Modular" as this configuration worked well for me in the past and is a general configuration that will work well for the purpose of this project. The config can be found at https://github.com/olafhartong/sysmon-modular. Once both are installed, I ran the following command which tells sysmon to use the following configuration file. 
![Screenshot 2024-08-21 220233](https://github.com/user-attachments/assets/bba6844a-d857-45ef-bfe9-94ba88d387ba) 

Setting a new ruleset for Splunk Universal Forwarder to follow when sending info to the Splunk server: 
For this step in my configuration, I referred to a youtube user by the name MyDFIR (https://www.youtube.com/@MyDFIR) for obtaining and applying the new inputs.conf file. The origional inputs.conf file resides in the C:\Program Files\SplunkUniversalForwarder\etc\system\default directory. Within this file a disclaimer is shown warning you to not edit the file. I made my own inputs.conf file in the C:\Program Files\SplunkUniversalForwarder\etc\system\local directory and used the config MyDFIR provides in the video. After the configuration file has been updated, I restarted the Splunk Universal Forwarder service in the services tool in windows.

![Screenshot 2024-08-25 211048](https://github.com/user-attachments/assets/249b296e-c86c-4749-88d8-3e426312ffee)
![Screenshot 2024-08-21 230556](https://github.com/user-attachments/assets/668c37dd-8e7b-4e9a-96da-9fd26960a7f4) 

Creating a new index in the Splunk Manager: 
As seen in the image of the inputs.conf file I updated, each rule is pointing to an index named "endpoint". In order to view the data being captured, splunk needs to have an index by the name of "endpoint" to receive and aggregate the data. This new index will then give me the ability to search and analyze the data for security related events occurring on the machines.

![Screenshot 2024-08-26 001037](https://github.com/user-attachments/assets/586ca6a0-9766-48b5-96d1-de68a3a326fc)
Once the index is created, I enabled the ability to recieve the data meant for the endpoint index by going to settings>Forwarding and receiving>Configure receiving and I set the listening port to 9997. This matches the port I specified in the setup of Splunk Universal Forwarder earlier. In a real-world environment, this port should be changed as this is the default port Splunk uses for the recieving index. The common knowledge of port 9997 being used can give an attacker a possible attack vector to conduct their attack. It best to change this default port number to make it more difficult for an attacker to gain information about the network infastructure. 
