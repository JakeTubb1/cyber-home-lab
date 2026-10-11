# Goal: Create and understand SOC environments and how they help protect systems while also understanding how to simulate real-world attacks. Also test detection capabilities of Wazuh and Security Onion.
### Tools: VMware, Kali, Windows Server 2022, Wazuh, Security Onion
Connect everything on a virtual network where I can attack, detect and respond!

## Inventory and Architecture:
 VM Name + RAM dedication| Role | OS | IP Address <br>
Security Onion 16gb| NSM/IDS | Security Onion OS | 192.168.253.100 <br> 
Kali and Wazuh 4gb | Attacking and XDR/SIEM | Kali Linux | 192.168.253.134 <br>
Windows Server 2022 4gb | Target Machine | Windows 10 | 192.168.253.101**fix***   <br>


![network diagram](./Images%20for%20MD/network%20diagram.png)


## Implementation:
### Constraints & Design Decisions: <br>
#### Why I put wazuh on my attack machine: <br>
In a real-world context, this would not be feasible as a production environment requires a dedicated machine for event logs, but as I am working with limited resource (32gb RAM) to keep my VMs from performance deterioration, I decided to stack Wazuh alongside my attack toolkit (Kali VM). This keeps my vms running smoothly and also helps with simplicity in this context. <br>

#### Why used a dynamic ip instead of a static ip address: 
When setting up this lab I opted for a dynamic ip address as I knew I would be using this lab at multiple different locations. From my research I understood that static ips are generally tied to one location so when my physical machine is on a different Wi-Fi then my lab may not work. Business tend to use static ip address. They do this for ease of hosting servers and simplified remote access. It teams only have to set up a firewall then create a ‘white list’ of external ip address that are allowed to connect limiting outsider threats but as they don’t change it makes it easier for adversaries to test and attack over time. While businesses prefer static ips for these reasons I opted for dynamic for portability. <br>

# Detection:
1.	Nmap
For my first test within my SOC I started with a basic nmap scan (Nmap -A -T4 192.168.253.101). -A stands for aggressive, I used this to make sure security onion picked it up I also used -T4 to speed it up making it even more likely the security onion detects it.  This is used for OS detection, version detection, script scanning, and traceroute. <br>
### How this maps to the cyber kill chain and MITTRE ATTACK: <br>
This nmap scan falls into the recognisance stage of the cyber kill chain. It maps to the MITTRE ATTACK framework via TA0043 (Reconnaissance) and the sub technique is T1595 (Active Scanning). There is no specific mitigation for this but companies can minimise the amount of sensitive data available. <br>

![nmap scan](./Images%20for%20MD/nmap%20scan.png) <br>
 
### Wazuh:
 ![Wazuh for nmap](./Images%20for%20MD/wazuh%20for%20nmap.png) <br>
Wazuh did not pick up the nmap scan which shows the importance of running endpoint detection (EDR)and network intrusion detection (NIDS)
The one alert is just a notification saying the vm containing wazuh is online.

### Security onion:
 
 ![kali in secon](./Images%20for%20MD/kali%20linux%20in%20secon.png)

 ![secon nmap](./Images%20for%20MD/secon%20for%20nmap.png) <br>
Counts: <br>
In the screen shot we can see the count for Nmap scripting is very high (37) compared to inbound traffic to The SQL ports which is 2. This shows how noisy automated scripting is rather than individual port checks. <br>
2022973: <br>
As we can see even before the nmap scan started we could see the security onion picked up that there is a Kali Linux host on the network. This shows security onion can pick up potentially high-risk operating system before an attack begins, it also means there is an attacker already plugged into your network. <br>

2002910 + 2010935-39:<br>
Suricata detected specific probing for common services. This shows the attacker is moving from general network discovery to Service Enumeration, which is where the attacker is trying to establishes a connection with the host to find out what services are running and what version they are.<br>

20244364: <br>
This shows that security onion has picked up and nmap scan. It has done this by reading the packets that nmap sent and security onion knows what these packets looks like and detected them. <br>

2.	 Brute force <br>
The command: <br>
crackmapexec winrm 192.168.253.137 -u /home/kali/Desktop/Usernames -p /home/kali/Desktop/Passwords <br>
How this maps to the cyber kill chain and MITTRE ATTACK <br>
I am now developing the attack on the windows server using a brute force password cracking tool. This maps to the mitre attack framework via Credential access (TA0006) the technique of Brute Force (T1110) and the sub technique Password Guessing (T1110.001). A mitigation for this is M1027- NIST guidelines when creating password policies. The requires passwords to have a minimum of 15 characters and permits up to 64. Also banning frequent password changes as user will pick weaker passwords each time. Changing this stops passowrd guessing from tools like and others. It maps to reconnaissance, delivery and exploitation within Lockheed martin’s cyber kill chain. <br>


![crackpipe](./Images%20for%20MD/crackpipexec.png)

### Wazuh
![wazuh crackpipe](./Images%20for%20MD/wazuh%20for%20crackpipe.png)

After running the brute force command my wazuh shows lots of level 5 alerts and some level 10 alerts. <br>

60122: <br>
Logon Failure - Unknown user or bad password (level 5)<br>
This is wazuh picking up a password and username guess i performed from my kali machine which failed. <br>
Level 5 = User Error (Low-Medium Severity)

60204: <br>
Multiple Windows Logon Failures (level 10)<br>
This shows that wazuh picked up the frequent logon attempts from my kali machine and flagged them as higher. than a normal failed attempt. <br>
Level 10 = Potential Attack (High Severity)


### Secuirty Onion
![secon crackpipe](./Images%20for%20MD/secon%20for%20crackpipe.png) <br>

There are no alerts here which show a brute force attack. <br>

### Wazuh Alerts Within Security Onion

![wazuh in secon crack](./Images%20for%20MD/wazuh%20log%20in%20secon%20for%20crackpipe.png) <br>

![level 10 wazuh secon crack](./Images%20for%20MD/level%2010%20alerts%20secon%20crackpipe.png)

After using the hunt feature in Security Onion i could see the same alerts as within wazzuh after specifically searching for "logon failure" and "Multiple Windows Logon Failures". Without wazuh their would be no way to have know this has happened. This shows the extreme importance of runinng a HIDS (Host IDS) and a NIDS (Network IDS). Without Wazuh i wouldnt have picked up the brute force attempt and with out Security Onion I wouldnt have picked up the nmap scan. 

## Maintenance
In case of VM accidents, I have used snapshots to save certain states of VMs as I go along. This allows me to revert back to old VM states if something goes wrong or a system gets broken. <br>



# Improvement
After spending a week in industry and talking to many cyber secuirty proffesionals i have been suggested ideas on what to add to help devlop my real world aplity to read logs and build a more complex homelab which is similar to real wrld senarios


 VM Name + RAM dedication| Role | OS | IP Address <br>
Security Onion 16gb| NSM/IDS | Security Onion OS | 192.168.253.100 <br> 
Kali and Wazuh 4gb | Attacking and XDR/SIEM | Kali Linux | 192.168.253.134 <br>
Windows Server 2022 4gb and Sysmon| Target Machine | Windows 10 | 192.168.253.101**fix***   <br>


## Sysmon install
For sysmon I am using a config file off of github. This config file is from SwiftOnSecuirty (https://github.com/SwiftOnSecurity/sysmon-config) <br>
What swift on Secuirty provides: <br>
| Event ID | Event Type | Purpose |
| :--- | :--- | :--- |
| 1 | Process Creation | Logs all new process launches with command line arguments |
| 3 | Network Connection | Captures outbound TCP/UDP connections |
| 7 | Image Loaded | Tracks DLL/module loading into processes |
| 8 | CreateRemoteThread | Detects process injection attempts |
| 10 | ProcessAccess | Monitors inter-process access (handles opened) |
| 11 | FileCreate | Logs file creation events in monitored directories |
| 12, 13, 14 | Registry Events | Captures registry object creation and modification |
| 15 | FileCreateStreamHash | Tracks alternate data streams (ADS) |
| 17, 18 | Pipe Events | Monitors named pipe creation and connections |
| 19, 20, 21 | WMI Events | Logs WMI activity |
| 22 | DNS Query | Captures DNS lookups |
| 23 | FileDelete | Tracks file deletion events |
| 26 | ClipboardChange | Monitors clipboard operations |
| 27 | ProcessTampering | Detects process memory tampering |
| 28 | HardwareChanged | Logs hardware detection events |





### Flow from sysmon to wazuh
Sysmon (running on Windows)<br>
        ↓<br>
Windows Event Log (Microsoft-Windows-Sysmon/Operational)<br>
        ↓<br>
Wazuh Agent (on Windows VM) reads Event Log<br>
        ↓<br>
Wazuh Manager (on Kali) receives and processes logs<br>
        ↓<br>
Wazuh Dashboard shows alerts<br>

## 3 specific Windows event with Sysmon IDS<br>

![powershell admin sysmon](./Images%20for%20MD/1sysmon.png)<br>


To generate these logs i opended power shell as adminstrator.<br>

These logs show:<br>

What application was ran:<br>
In this case it was powershell.exe<br>

Who ran it:<br>
Adminstartor<br>

Parent:<br>
explorer.exe (PID 2096)<br>

PID:
6052 (powershell)<br>

Integrity:<br>
high (Elevated)<br>

These 2 events alone can be harmless but from this stage attackers could run scripts within powerhsell to potentially find and get valuable information to them.<br>


![powershell whoami sysmon](./Images%20for%20MD/2sysmon.png)<br>
This is potentially a attacker who has acess tothis machine and is try to figure out what role the person has<br>

From the event properties we can see the person on this mahcine has run whoami in powershell. We can see this from:<br>

Parent:<br>
ParentProcessId: 6052 (PowerShell)<br>
ParentImage: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe<br>
ParentCommandLine: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"<br>

CommandLine: "C:\Windows\system32\whoami.exe"<br>

![powershell ipconig /all sysmon](./Images%20for%20MD/3sysmon.png)<br>
The attacker is now looking at the ip adreess of the machine in powershell:<br>

We can see they have run ipconfig /all from:<br>
"C:\Windows\System32\ipconfig.exe" /all<br>


Parent:<br>
ParentProcessId: 6052 (PowerShell)<br>
ParentImage: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe<br>
ParentCommandLine: "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"<br>



# Ai SOC Helper - Wazuh, OpenSearch Indexer and Ubuntu AI Server
Windows Server VM<br>
        ↓<br>
     Sysmon<br>
        ↓<br>
   Wazuh Agent<br>
        ↓<br>
 Kali Linux Wazuh Server<br>
       ↓<br>
 Wazuh Manager<br>
        ↓<br>
 Wazuh Indexer (OpenSearch)<br>
        ↓<br>
 SSH Tunnel<br>
        ↓<br>
 Ubuntu AI Server<br>
        ↓<br>
 Python Integration<br>
        ↓<br>
 Ollama AI Model<br>

## What I have learned:

## What I Have Learned

Throughout developing this home lab, I have gained practical experience in building and managing a virtualised cybersecurity environment using VMware.

I have developed a better understanding of how SIEM, HIDS and NIDS technologies work together to detect suspicious activity. By performing controlled Nmap scans and password-guessing attacks, I observed how Security Onion and Wazuh detect different types of threats and the importance of combining network and endpoint monitoring.

Configuring Sysmon has improved my understanding of Windows event logs, process creation, and how attackers' activities can be investigated through system telemetry.

I have also gained experience troubleshooting network connectivity, configuring security tools and analysing alerts to understand why certain activities are detected while others are not.

Overall, this project has helped me develop practical cybersecurity skills beyond theoretical knowledge and has provided a foundation for my ongoing AI-assisted SOC dissertation.


#### References:
https://www.lightspeed.co.uk/5-benefits-of-static-ip-address-for-business <br>
 Accessed: 2026-05-06 <br>
https://attack.mitre.org/ <br>
 Accessed: 2026-05-06 <br>
 <br>
Glossary:<br>
IDS- Intrusion detection system <br>
NSM- Network security monitoring <br>
XDR- End point <br>
SIEM- security information and event management <br>
Service Enumeration- Trying to establishes a connection with a target system to identify running services


