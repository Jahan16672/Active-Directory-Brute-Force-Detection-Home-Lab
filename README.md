# Active Directory Brute Force Detection Home Lab

## Overview

This project establishes a comprehensive virtual testing environment using Oracle VM VirtualBox, featuring Windows 10, Kali Linux, Windows Server, and Ubuntu Server virtual machines. The setup includes network configuration for VM intercommunication through IP addressing and NAT networking. The security framework incorporates Splunk Server for log monitoring and analysis, Universal Forwarder for data transmission, and Sysmon for comprehensive endpoint surveillance. Attack simulation utilizes Crowbar for credential-based attacks, Atomic Red Team (ART) for security validation testing, and Splunk for detailed log examination. Windows systems are integrated into an Active Directory infrastructure with Remote Desktop functionality enabled. PowerShell automation streamlines various administrative tasks, providing practical experience with cybersecurity tools and methodologies in a secure, isolated environment.

**Estimated Duration:** 5-6 hours

## Lab Architecture Overview

![image](https://github.com/user-attachments/assets/9ca35a47-f675-4892-8c6f-8df815654684)

*Ref 1. Active Directory Lab Architecture Diagram*

## Purpose and Goals

This laboratory exercise provides comprehensive hands-on experience in establishing a virtualized cybersecurity testing and analysis environment. Through the creation and configuration of multiple virtual machines including Windows 10, Kali Linux, Windows Server, and Ubuntu Server, participants develop essential skills in network architecture, security tool deployment (Splunk, Sysmon), system monitoring, and penetration testing methodologies (utilizing Crowbar for authentication attacks). The integration of Windows systems into an Active Directory domain structure, combined with Remote Desktop configuration, enhances the learning experience. PowerShell scripting facilitates task automation throughout the project. The laboratory provides participants with practical exposure to cybersecurity principles, tools, and attack/defense techniques within a controlled, isolated environment.

## Technical Competencies Developed

- Virtual machine deployment and management (Windows 10, Kali Linux, Windows Server, Ubuntu Server) using Oracle VM VirtualBox
- Network configuration including IP addressing and NAT Network setup for virtual environments
- Network connectivity diagnosis and resolution (ping utilities, DNS configuration)
- Splunk Server deployment and Universal Forwarder configuration
- Sysmon implementation for comprehensive endpoint monitoring
- Crowbar utilization for authentication-based attack simulation
- Atomic Red Team (ART) deployment for security control validation
- Security event analysis (focusing on event codes 4625, 4624) within Splunk environment
- Windows domain integration and Active Directory management
- Remote Desktop Protocol (RDP) configuration and security
- PowerShell scripting and automation (Invoke-WebRequest, Set-ExecutionPolicy)

These competencies encompass virtualization technologies, network engineering, software deployment and configuration, cybersecurity tools and methodologies, operating system administration, documentation practices, scripting and automation, troubleshooting techniques, Active Directory administration, and endpoint security monitoring.

## Tools Used

- **Oracle VM VirtualBox Manager:** Primary hypervisor for virtual machine creation and administration
- **Splunk Server:** Centralized logging platform for security event analysis and monitoring
- **Splunk Universal Forwarder:** Agent for log data collection and transmission to Splunk
- **Sysmon:** Advanced system monitoring tool for Windows endpoint surveillance
- **Crowbar:** Brute-force attack tool for credential security testing
- **Atomic Red Team (ART):** Framework for security control testing and validation
- **PowerShell:** Command-line interface and scripting environment for automation
- **Microsoft Windows Event Logs:** Primary data source for security monitoring within Splunk
- **Windows Server 2022:** Server operating system hosting Active Directory Domain Services
- **Ubuntu Server:** Linux distribution serving as the Splunk server platform
- **Microsoft Windows 10:** Client operating system functioning as the target endpoint
- **Kali Linux:** Specialized penetration testing distribution used for attack simulation

## Implementation Guide

### Phase 1 - Virtual Machine Deployment

#### 1. Oracle VM VirtualBox Installation

Access [VirtualBox Downloads](https://www.virtualbox.org/), obtain the appropriate version for your system, and complete installation including all required dependencies.

#### 2. Windows 10 Deployment

1. Visit [Microsoft Windows 10 Download](https://www.microsoft.com/en-ca/software-download/windows10)
2. Obtain the "Create Windows 10 installation media" tool
3. Select "Download tool now"
4. Choose "Create installation media (USB flash drive, DVD, or ISO file) for another PC"
5. Select "ISO file" option and save to desired location
6. Within Oracle VM VirtualBox Manager, select "Add" to initialize new VM creation
7. Assign appropriate name, select the downloaded Windows 10 ISO image
8. Allocate 4096MB RAM, assign 1 CPU core, create 50GB virtual hard disk
9. Complete setup and launch the VM
10. Proceed through installation wizard, select "Custom: Install Windows only (advanced)" option
11. Allow Windows 10 installation to complete

#### 3. Kali Linux Deployment

1. Obtain Kali Linux from [Kali.org](https://www.kali.org/)
2. Download the pre-built VM version
3. Install 7-zip from [7-Zip.org](https://www.7-zip.org/)
4. Extract the Kali Linux archive using 7-zip
5. Import the extracted VM into Oracle VM VirtualBox Manager
6. Initialize the virtual machine

#### 4. Windows Server Deployment

1. Download Windows Server 2022 ISO from [Microsoft Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
2. Complete the required form and download the "64-bit edition"
3. Create new VM within Oracle VM VirtualBox Manager using the ISO
4. Allocate 4096MB RAM, assign 1 CPU core, create 50GB virtual hard disk
5. Finalize configuration and start the VM
6. Select "Install now", choose "Windows Server 2022 Standard Evaluation (Desktop Experience)"
7. Configure custom settings, establish administrator password, and complete installation

#### 5. Ubuntu Server Deployment

1. Navigate to [Ubuntu Server](https://ubuntu.com/server)
2. Access Products > Ubuntu Server and download Ubuntu Server (this laboratory utilizes version 22.04.4 LTS)
3. Create new VM in Oracle VM VirtualBox Manager with the ISO image
4. Allocate 8192MB RAM, assign 2 CPU cores, create 100GB virtual hard disk
5. Complete setup and start the VM
6. Select "Try or Install Ubuntu Server"
7. Proceed through configuration screens selecting "Done" and "Continue" as appropriate
8. Complete the user information form and continue installation process
9. Execute reboot when prompted (error messages are expected during this process)
10. After successful reboot, authenticate and execute:
    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    ```
11. Upon completion, press Enter to continue

**Summary:** At this point, you should have Oracle VM VirtualBox Manager configured with four operational VMs: Windows 10, Kali Linux, Windows Server, and Splunk Server.

### Phase 2 - Network Architecture Configuration

#### 1. Communication Infrastructure Setup

1. Navigate to Tools > Network > NAT Networks > Create
2. Specify network name and IPv4 Prefix (this laboratory uses `192.168.10.0/24`)
3. Apply configuration
4. For each VM, access Settings > Network, modify "Attached to: NAT Network" and assign the previously created NAT Network name
5. Launch the Splunk VM and execute:
   ```bash
   sudo nano /etc/netplan/00-installer-config.yaml
   ```
6. Modify the configuration to match the following structure:
   ```yaml
   network: 
     ethernet:
       enp0s3:
         dhcp4: no
         addresses: [192.168.10.10/24]
         nameservers:
             addresses: [8.8.8.8]
         routes:
             - to: default
               via: 192.168.10.1
     version: 2
   ```
7. Execute `sudo netplan apply` to implement changes
8. Run `ip a` to verify IP address assignment to `192.168.10.10/24`
9. Validate connectivity by executing `ping google.com`

#### 2. Splunk Installation and Configuration

1. Access [Splunk.com](https://www.splunk.com/) and download the free trial of Splunk Enterprise for Linux (.deb package)
2. Return to the Splunk VM and execute:
   ```bash
   sudo apt-get install virtualbox-guest-additions-iso
   ```
3. Navigate to Devices > Shared Folders > Shared Folders Settings > Create new Shared Folder
4. Browse to the directory containing the Splunk installation file, enable all three configuration options, and proceed
5. Restart the virtual machine using `sudo reboot`
6. Execute the following commands:
   ```bash
   sudo apt-get install virtualbox-guest-utils
   ```
7. Reboot again, then run:
   ```bash
   sudo adduser <your username> vboxsf
   ```
8. Create a new directory:
   ```bash
   mkdir share
   ```
9. Execute:
   ```bash
   sudo mount -t vboxsf -o uid=1000,gid=1000 <directory name containing .deb file> share/
   ```
10. Verify successful completion with `ls -la` - "Share" should appear highlighted
11. Navigate to the share directory and install Splunk:
    ```bash
    cd share/
    ls -la
    sudo dpkg -i splu<tab for auto-completion>
    ```
12. Navigate and start Splunk:
    ```bash
    cd /opt/splunk/
    ls -la
    sudo -u splunk bash
    cd bin/
    ./splunk start
    ```
    Press 'q' followed by 'y' and [ENTER] to continue
13. Complete configuration:
    ```bash
    exit
    cd bin
    sudo ./splunk enable boot-start -user splunk
    ```

#### 3. Windows Target Machine Configuration

1. Access the Start Menu and search for "About" > Rename this PC
2. Assign desired name (this laboratory uses "target-PC")
3. Restart the system
4. Launch Command Prompt, execute `ipconfig` and note the current IPv4 Address
5. Access the network icon in the system tray, right-click > Open Network & Internet Settings > Change adapter options
6. Right-click the network adapter > Properties > Double-click "Internet Protocol Version 4 (TCP/IPv4)"
7. Select "Use the following IP address" and configure:
   - IP Address: `192.168.10.100`
   - Subnet mask: `255.255.255.0`
   - Default gateway: `192.168.10.1`
   - Preferred DNS server: `8.8.8.8`
8. Execute `ipconfig` again to verify the new IP configuration

**Splunk Universal Forwarder Installation:**

1. With the Splunk server operational, access it through the target machine's browser: `192.168.10.10:8000`
2. On the target machine, visit [Splunk.com](https://www.splunk.com/) and navigate to Products > Free Trials & Downloads > Universal Forwarder
3. Download the appropriate version and execute the MSI file
4. Configure basic information without creating a password
5. Skip the deployment server configuration
6. Set the Receiving Indexer IP/port to `192.168.10.10:9997`
7. Complete installation

**Sysmon Installation:**

1. Install Sysmon by accessing [Microsoft Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
2. Navigate to [Sysmon Config](https://github.com/olafhartong/sysmon-modular)
3. Scroll down and select `sysmonconfig.xml`, click "raw" and save the file
4. Extract the Sysmon archive, copy the extraction directory path
5. Launch PowerShell as administrator and navigate to that directory
6. Execute:
   ```powershell
   .\Sysmon64.exe -i ..\sysmonconfig.xml
   ```
7. Accept the license agreement

**Critical Configuration Step:**

1. Navigate to File Explorer > Local Disk (C:) > Program Files > SplunkUniversalForwarder > etc > system > local
2. Launch Notepad as administrator and input the following configuration:
   ```ini
   [WinEventLog://Application]
   index = endpoint
   disabled = false

   [WinEventLog://Security]
   index = endpoint
   disabled = false

   [WinEventLog://System]
   index = endpoint
   disabled = false

   [WinEventLog://Microsoft-Windows-Sysmon/Operational]
   index = endpoint
   disabled = false
   renderXml = true
   source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
   ```
3. Save this configuration as "inputs.conf" in the local folder, ensuring "All Files" is selected as the file type
4. Launch Services as administrator, locate and double-click SplunkForwarder
5. Access the Log On tab and select "Local System Account"
6. Right-click Splunk Forwarder and select restart
7. Navigate to `192.168.10.10:8000` and authenticate
8. Access Settings > Indexes > New Index, name it "endpoint", and save
9. Navigate to Settings > Forwarding & receiving > Configure receiving > New Receiving Port, set port to 9997
10. Access Apps > Search & Reporting and search for `index=endpoint`

#### 4. Windows Server Configuration

Follow the same network configuration and Splunk setup steps as the Windows Target Machine, but with these differences:

- Computer name: "ADDC01"
- IP Address: `192.168.10.7`
- All other settings remain the same

**Summary:** When accessing your Splunk server from either Windows machine, you should observe under "Selected fields" > "Host" two Values representing your VMs: TARGET-PC and ADDC01.

### Phase 3 - Active Directory Implementation and Domain Management

#### 1. Active Directory Domain Services Configuration on Windows Server

1. On the Windows server, launch Server Manager
2. Navigate to Manage > Add Roles and Features
3. Select Next > Next, then check "Active Directory Domain Services" > Add Features
4. Continue through the wizard until you can select Install
5. Upon receiving the message "Configuration required. Installation succeeded on ADDC01", proceed to the next steps
6. Locate the notification flag icon at the top of the window and select "Promote this server to a domain controller"
7. Select "Add a new forest" since we're establishing a completely new domain
8. Assign the name "demodomain.local"
9. On the subsequent page, maintain all default settings and create an administrator password
10. Continue through the wizard until the Configuration Wizard validates prerequisites, then select Install
11. This process will sign you out and restart the server

**User Creation:**

1. When the machine restarts, you should see "DEMODOMAIN\Administrator" as the authenticated user
2. In Server Manager, navigate to Tools > Active Directory Users and Computers
3. Right-click demodomain.local > New > Organizational Unit > Name: "IT"
4. Right-click within this new OU > New > User
   - First name: "Jenny"
   - Last name: "Smith"
   - Full name: "Jenny Smith"
   - User logon name: "jsmith"
5. Establish a password and complete user creation
6. Create an additional OU named "HR" and establish another user with different credentials

#### 2. Windows 10 Domain Integration

1. From the target machine, access Windows search and enter "Advanced System Properties" > Computer Name > Change
2. Select "Domain:" and enter "DEMODOMAIN.LOCAL"
3. Access the network icon in the system tray, right-click > Open Network & Internet Settings > Change Adapter Options
4. Right-click adapter > Properties > Double-click "Internet Protocol Version 4 (TCP/IPv4)"
5. Modify the Preferred DNS server to `192.168.10.7`
6. In Command Prompt, execute `ipconfig /all` to verify DNS Servers is configured as `192.168.10.7`
7. Return to Computer Name/Domain Changes and select "OK"
8. Authenticate with administrator credentials and restart the machine
9. When accessing the login screen, select "Other User" and verify the domain authentication is pointing to "DEMODOMAIN"
10. Authenticate using the credentials of a user created earlier within the IT or HR organizational units

### Phase 4 - Kali Linux Attack Simulation Against Target Machine

#### 1. Network Configuration

1. Power on the Kali Linux VM and authenticate
2. From the desktop, right-click the network connections icon in the top-right corner
3. Edit Connections > Wired connection 1 > Edit selected connection > IPv4 Settings
4. Set Method to "Manual" > Add:
   - Address: `192.168.10.250`
   - Netmask: `24`
   - Gateway: `192.168.10.1`
   - DNS servers: `8.8.8.8`
5. Save, disconnect, then reconnect to the network
6. Launch terminal and execute `ip a` to verify the configuration changes
7. Validate connectivity:
   ```bash
   ping google.com
   ping 192.168.10.10
   sudo apt-get update && sudo apt-get upgrade -y
   ```

#### 2. Attack Preparation

1. Execute:
   ```bash
   mkdir ad-project
   sudo apt-get install -y crowbar
   ```
2. Navigate to wordlists:
   ```bash
   cd /usr/share/wordlists
   ls
   sudo gunzip rockyou.txt.gz
   cp rockyou.txt ~/Desktop/ad-project
   cd ~/Desktop/ad-project
   head -n 20 rockyou.txt > passwords.txt
   ```

#### 3. Remote Desktop Protocol Activation

1. On the target machine, search for "This PC" > Properties > Advanced System Settings > Remote
2. Check "Allow remote connections to this computer" > Select users > Add
3. Enter the usernames created earlier such as "jsmith" and "tsmith" > Apply

#### 4. Attack Execution

1. Return to the Kali Linux machine
2. Execute:
   ```bash
   crowbar -h  # Review tool options and usage
   crowbar -b rdp -u tsmith -C passwords.txt -s 192.168.10.100/32
   ```
3. If a password within `passwords.txt` matches the password assigned to the specified user, you will receive a success message displaying the compromised credentials

#### 5. Attack Analysis Using Splunk

1. Access all endpoint activity using `index=endpoint`
2. For specific account activity, append the username such as `tsmith`
3. When scrolling through results and selecting EventCode, you'll observe 3 event types
4. **Value 4625** is significant because it occurred 20 times - this represents [failed logon attempts](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4625)
5. All occurrences happen within approximately the same timeframe, indicating brute force attack activity
6. **Event code 4624** indicates [successful logon events](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624)
7. Expand this event by clicking "Show all x lines" to view the source of this authentication

#### 6. Target Machine Testing Using Atomic Red Team (ART)

1. Launch PowerShell as administrator
2. Execute:
   ```powershell
   Set-ExecutionPolicy Bypass CurrentUser
   ```
   Respond with Y
3. Click the notification arrow in the bottom-right corner
4. Access Windows Security > Virus & threat protection > Manage Settings > Add or Remove Exclusions > Add an exclusion > Folder > This PC > Local Disk (C:)
5. Return to PowerShell and execute:
   ```powershell
   IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
   Install-AtomicRedTeam -getAtomics
   ```
   Respond with Y
6. Navigate to the (C:) drive > AtomicRedTeam > Atomics
7. Reference [MITRE ATT&CK](https://attack.mitre.org/) to view adversary tactics and techniques
8. Execute:
   ```powershell
   Invoke-AtomicTest T1136.001
   ```
   T1136 represents a persistence technique for local account creation
9. After executing this test and checking Splunk, observe that no events appeared for the NewLocalUser creation
10. This discovery reveals a significant gap in our monitoring capabilities
11. Continue testing with additional techniques as desired

## Project Conclusion

You have successfully completed this comprehensive walkthrough! With the conclusion of this detailed guide, you should have established successful setup and communication between four Virtual Machines:

- **Windows 10 Target Machine**
- **Kali Linux Attacker Machine** 
- **Splunk Server**
- **Windows Server**

You should be capable of:
- Viewing security events through Splunk
- Testing defensive capabilities using Crowbar brute force attacks
- Validating security controls with Atomic Red Team
- Managing Active Directory environments
- Analyzing security logs and identifying attack patterns

## Network Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Windows 10    │    │   Kali Linux    │    │ Windows Server  │    │ Ubuntu Server   │
│   Target-PC     │    │   Attacker      │    │    ADDC01       │    │ Splunk Server   │
│ 192.168.10.100  │    │ 192.168.10.250  │    │ 192.168.10.7    │    │ 192.168.10.10   │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │                       │
         └───────────────────────┼───────────────────────┼───────────────────────┘
                                 │                       │
                    ┌─────────────────────────────────────┐
                    │        NAT Network                  │
                    │      192.168.10.0/24               │
                    │    Gateway: 192.168.10.1           │
                    └─────────────────────────────────────┘
```

## Key Learning Outcomes

This project demonstrates the value of Atomic Red Team for organizational security validation and highlights the importance of comprehensive monitoring in cybersecurity environments. The hands-on experience with attack simulation and defense provides practical insights into real-world cybersecurity scenarios.

## Acknowledgments

Special acknowledgment to [MyDFIR on YouTube](https://www.youtube.com/@MyDFIR) for the foundational tutorial content. This project provided valuable learning opportunities to apply theoretical knowledge and gain practical cybersecurity experience.

## Resources

- [Oracle VM VirtualBox](https://www.virtualbox.org/)
- [Splunk Enterprise](https://www.splunk.com/)
- [Microsoft Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [MyDFIR YouTube Channel](https://www.youtube.com/@MyDFIR)

---

*This project is for educational purposes only. Always ensure you have proper authorization before conducting any security testing.*
