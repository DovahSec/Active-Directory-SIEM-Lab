# Active-Directory-Lab
Active Directory Domain to simulate managing user accounts and monitoring activity within the domain. 

This purpose of this project was to expose myself to hands-on experience with IT administration and build a splunk instance to gather telemetry attached to the domain and target machine. I wanted to extend the project beyond the typical AD deployment and implement an attacking machine within the network to demonstrate how sysmon and splunk work together to gather and deliver security events I configure within an input.conf file. I will also demonstrate how this configuration can be used to conduct vulnerability assesments using AtomicRedTeam. I plan to have another project that gets deeper into the configuration of the Active Directory server and the users within the domain to simulate a real-world environment. This project mainly focuses on the presence of a SIEM within the domain and how it can be used to identify important security events. I will edit this README once the project is finished and create a new repository for it.

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

Utilizing the search functionality in Splunk manager: 
Now that my index has been created and I verify data is flowing into it, I can use the search function to query specific data from the collection. As an example, I can input the following search query and receive data within the past seven days from the endpoint index. 
![Screenshot 2024-08-26 110701](https://github.com/user-attachments/assets/4461ea0b-24bd-408a-b676-11388cd2c462)

Configuring Active Directory Domain: 
First step on ADDC01 is to select the instllation type "Role-based or feature-based installation". This will allow me to create and manage user roles for this lab and future projects using this domain controller. Next is to promte the server to a Domain Controller. I named the root domain as dovahsec.local, the .local is the top-level domain name for the domain so when other machines attempt to reach the server, the local DNS queries can respond properly. 
![Screenshot 2024-08-26 113058](https://github.com/user-attachments/assets/5311b09f-df64-4325-8716-d923a21db3e0) ![Screenshot 2024-08-26 113407](https://github.com/user-attachments/assets/7b41b457-b4ed-4b9e-859e-f10c89a24bf0)

Creating the users for the lab and joining the domain on the target machine: 
I created two Organizational Units that will be used in this lab, IT and HR. bdaniels is the user within IT and ortega is the user within HR. I then went to the target machine and joined the dovahsec.local domain in advanced system settings. When I selected ok, an error splash screen is shown:
![Screenshot 2024-08-23 163417](https://github.com/user-attachments/assets/0b2dc70c-0cf3-432a-a087-3a26e5714aec)This error ties back to my earlier comment about the DNS server being able to resolve dovahsec.local domain properly. The current DNS server (8.8.8.8) does not know how to resolve dovahsec.local. To fix this issue, I located the IPv4 settings in the ethernet properties and changed the DNS server from 8.8.8.8 to the domain controller for dovahsec.local (192.168.10.7). After the change I ran ipconfig /all to verify the DNS server has changed properly. I can then go back and join the dovahsec.local domain after endering the domain administrator's credentials. In a real-world environment, I would desegnate a special role with custom users that specifically handles the operation of joining machines to the domain. This removes the risk of the administrators credentials falling into the wrong hands while joining many different machines to the domain. 
![DNSchangeandDomainsuccess](https://github.com/user-attachments/assets/15bfc12e-f485-4b80-bb93-b44d52f8c8b1)

Configuring the attacker's Kali Linux machine to conduct the attack: 
For the initial configuration steps, I will list off the operations I conducted until I get to a major talking point to save time and keep this repo from being too long. 

Changed static IP to 192.168.10.250/24, Gateway to 192.168.10.1, DNS server to 8.8.8.8, refresh the ethernet adapter to reflect the changes made. $ ip a to verify configuration.

"sudo apt-get update && sudo apt-get upgrade -y" 

"~/Desktop  $ mkdir ad-project"

"sudo apt-get install -y crowbar" 

"cd /usr/share/wordlists" 

"sudo gunzip rockyou.txt.gz" 

"cp rockyou.txt ~/Desktop/ad-project" 

For this lab, rockyou.txt is too large for what I need on this machine, so I will limit the wordlist to 20 lines and save the new list to a new txt file called passwords.txt. "head -n 20 rockyou.txt > passwords.txt" 

This list of 20 lines from rockyou.txt does not accurately represent how an attacker would actually conduct their attack. Obtaining relevant passwords would come from various types of reconnaissance. To simplify this lab, Im using the 20 lines from rockyou.txt file and adding the passwords of the two users I created in AD. 

Crowbar is a tool mainly used to brute fore access to remote access services such as OpenVPN, RDP, SSH Private Keys, and VNC Keys. For my lab I will be utilizing the RDP capabilities on the Windows 10 target machine as my attack vector. I first need to enable RDP on the target machine for crowbar to have a service to target. 

On the target machine: PC > Properties > Advanced System Settings > Remote > now I add bdaniels and tortega to the users able to connect to the machine via RDP. Now I can go back to the Kali Linux machine and begin the brute force attack. Following the -h output to help choose what options I need to select, I customize and run the command to begin the attack 

![Screenshot 2024-08-26 132912](https://github.com/user-attachments/assets/10d6e594-c941-4c3f-b5e3-5069ddcc0eac) ![Screenshot 2024-08-23 215210](https://github.com/user-attachments/assets/ad92650f-126c-41a5-bbfd-fe8033da04db) 
Taking note of the IP address following the -s option, I specified 192.168.10.100/32. This is the IP address of the target machine I am attempting to gain remote access of, the /32 narrows down the attack to just this IP address instead of another CIDR notation such as /24 that would scan more IP addresses than I need for this attack. The output from crowbar says the brute force attack of the user tortega is a success and we have obtained remote access to the target machine via RDP.

Now I can go to the Splunk manager and input a search query to narrow down my search and see if I gathered any telemetry from the event. The search results show a large number of events associated with the targeted user tortega. 
![Screenshot 2024-08-23 220456](https://github.com/user-attachments/assets/53930b08-489c-4f3a-9714-1baa65106b30) ![Screenshot 2024-08-23 220612](https://github.com/user-attachments/assets/726ea04d-1a96-4ff0-ba57-a522b9cd8f63)
On the left side of the window, I can open the "EventCode" field and splunk will tell me how many of the collected events are associated with each event code. A large majority of the valuse in the list are associated with the event code 4625. Using https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx I searched what the Event ID 4625 represents. This code is used for "An account failed to log on", with this information in mind, the timestamps of each event related to this code are logged at very close intervals. A total of 21 counts occurred within 2 seconds. Knowing that is well beyond the capabilities of a human manually entering passwords, it can be reasonably assumed that this is the result of a brute force attack. To make matters worse for the security team, there is 1 count of EventCode 4624 which represents "An account was successfully logged on". This is a clear IoC (Indicator of Compromise) from a successful brute force attack that has occurred and incident response should begin as soon as possible.
![Screenshot 2024-08-23 221310](https://github.com/user-attachments/assets/a33c4d5b-bdbf-4027-a90e-a98f6055d8ca)

Installing AtomicRedTeam to the target machine to test the current security configuration: Before I install AtomicRedTeam, I need to exclude the download location of AtomicRedTeam so windows defender does not flag the files associated with the download. Along with the exclusion, I need to bypass the execution policy for the current user so I can run scrits as an administrator in PowerShell to install the files I need. To exclude the location of the download from windows defender, head to Windows Security > Virus and Threat Protection > Manage Settings > Add an exclusion. The command to bypass the execution policy is as follows: "Set-ExecutionPolicy Bypass CurrentUser". 
The guide for installing AtomicRedTeam can be found on the Red Canary GitHub https://github.com/redcanaryco/invoke-atomicredteam/wiki/Installing-Invoke-AtomicRedTeam. 

PS C:\Windows\system32> IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics 

The Atomics installed with AtomicRedTeam add a folder containing a large list of Technique ID's that are in the form of "TXXXX.XXX". These technique ID's correlate to MITRE Attack Framework. I head over to https://attack.mitre.org/ to identify what each of the technique ID's represent. From the MITRE Attack Framework I can find an attack technique I want to test on the machine and see what gaps I have in my current security configuration. For my example, I chose to test T1136.001 which is for "Create Account: Local Account". To conduct the test, I input the command: "Invoke-AtomicTest T1136.001" the tool will then run a script that creates telemetry for me to analyze that correlates with the creation of a new local account. 
![Screenshot 2024-08-23 230943](https://github.com/user-attachments/assets/1176553a-9c33-48aa-bb27-0dfb9856e71d)
Referencing the output from PowerShell after the command was run, I included "NewLocalUser" in my search query. When I go to the Splunk manager and input the search "index=endpoint NewLocalUser" I am met with 12 events correlating to the creation of a user named "NewLocalUser". This can be seen in a good or a bad way depending on the scenario. Seeing the events come in live can be good as it shows the security team the activity they tested is being identified by Splunk and the team can insure the proper security controls are in-place and up-to-date. It could also be bad if this activity was overlooked in the past and the security team is just now discovering the possible vulnerability in their current security configuration. The security team would have to take action to determine if the gap in security has been previously exploited and work on a fix to prevent any future security incidents related to the discovery.

At this point all of the goals I set to achieve with this project have been met. I was able to configure all of the required servers and machines successfully and gather telemetry with the custom SIEM. I really enjoyed commiting my time to this project, as I felt I was gaining valuable experience I could use at an organization in the future. There were plenty of learning points for me to focus on which ended up giving me more ideas for future projects. With this project complete, I am going to start another set of tasks to configure the Active Directory domain in such a way to replicate how a real-world organization would have theirs configured. This will be done with policy configuration, implimentation of security controls, provisioning user permissions, and much more. I hope you enjoyed my documentation for this project and hope you follow me on the journey to my upcoming career!
