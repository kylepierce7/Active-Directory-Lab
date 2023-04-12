# Active-Directory-Lab <br/>
## AD DS home lab for study and penetration testing 
Home Active Directory Penetration Testing Lab

Software:
VMware Workstation 17 Player 
Oracle VM Virtualbox
Microsoft Server 2019
Windows 10 PRO
Ubuntu Linux 22.04 LTS
Kali Linux
Vulnhub: Mr Robot VM (possible infection)

## Phase 1: Implement Mr Robot VM from [Vulnhub.com](www.vulnhub.com) for CTF
Status: FAIL
### 
 Source: [Youtube.com | Cyber Studies| “How to build a secure hacking lab”](https://www.youtube.com/watch?v=lm_4hoe-K7U&t=578s)
	
 Create a DHCP server using the following Powershell script: 

`VBoxManage dhcpserver add --network=CyberStudyLab --server-ip=192.168.3.1 --netmask=255.255.255.0 --lower-ip=192.168.3.2 --upper-ip=192.168.3.254 –enable`
	
 Downloaded Mr Robot: 1 VM from VunlHub.com <br/>
•	Tried to confirm download integrity by comparing file hash: <br/>
•	Get-filehash -algorithm MD5 .\Downloads\mrRobot.ova <br/>
•	Windows Defender flagged the file as malicious.<br/>
•	Uploaded the fire to virus total for conformation; file size exceed upload limits on Virus Total and was unable to complete. Searched for the domain and identified malicious hosting listed on Virus Total. <br/>
•	Remediated, quarantined, deleted the file and preformed full system scan. <br/>
•	Alerted community of findings. <br/>

## Phase 2: Install and configure Windows Server 2019 for the purposes of studying Active Directory

Status: SUCCESS

 

Source: [Youtube.com | Josh Madakor | “How to setup a basic home lab running active directory”](https://www.youtube.com/watch?v=MHsI8hJmggI&t=1681s)
<h3> Server setup - <br/>
•	Download Windows Server 2019 Trial <br/>
•	Setup VM in Virtualbox but Windows wasn’t able to verify Microsoft trial license <br/>
•	Configured the VM to boot from the virtual CD drive and assigned the path to the .ios; successfully installed Server 2019 <br/>
•	Assigned password of: Password1 <br/>
•	Ctl+ALT+DEL to get to login screen <br/>
•	Install Guest Additions to optimize VM experience <br/>
o	Select “Insert Guest Additions” from Virtualbox <br/>
o	Installed Gest Additions CD in the VM Guest Host and restart VM <br/>
•	Assigned one NIC to NAT for internet connection; will disable this later, and one for internal network “CyberStudyLab” as configured in Phase 1. <br/>
•	After restart, confirmed that Guest Additions is functioning correctly <br/>
•	Check Network Adapter Options to confirm the two assigned NICs. Labeled them as: “_INTERNET_” and “X_INTERNAL_X” <br/>
•	Rename the PC; Right click on Start / System / Rename PC: “DC” <br/>
•	Assign IP via Network Settings / Change Adapter Options / “X_INTERNAL_X” / 172.16.0.1 / 255.255.255.0 / Gateway: <empty> / DNS: 127.0.0.1  <br/>
•	RE DNS, when Active Directory is installed it will act as its own DNS server so I have assigned the loopback address (127.0.0.1) to it <br/>
<h2> Active Directory Configuration - <br/>
<h3>
 •	Install Active Directory Domain Services (AD/DS) and create a domain <br/>
•	Server Manager / Dashboard / Add Roles and Features; follow the steps then add “Active Directory Domain Services” <br/>
•	Promote server to Domain Controller / Active Directory Domain Services Configuration Wizard / Add new forest / add Root Domain Name: “mydomain.com” <br/>
•	Computer will restart automatically when DNS configuration is completed. <br/>
<h2> Creating Admin Accounts - <br/> 
<h3>
 •	After restart and login, create a new domain admin account <br/>
•	Start / Administrative Tools / Active Directory Users and Computers  <br/>
•	Locate created domain “mydomain.com”; create a new Organizational Unit to store the new Administrator account we are creating. “_ADMINS” <br/>
•	In “_ADMINS”, create a new user (“a-kpierce”) ((a- represents an admin account)). <br/>
•	Edit user Properties / Member Of / Add / “domain admins” <br/>
•	Sign out and you can now log in under “Other user” with your account name and password <br/>
<h2> Configuring Remote Access and Network Address Translation - <br/>
<h3>
 •	Next step is to install Remote Access Server (RAS) and NAT. This will allow our Windows 10 client to access the internet through the domain controller. <br/>
•	Add Roles and Features / Remote Access / select “DirectAccess and VPN” and “Routing” <br/>
•	Tools / Routing and Remote access / DC Local / Configure and enable. If it doesn’t show up, close the window and try again. <br/>
•	Select “_INTERNET” and finish the install <br/>
<h2> Configuring the DHCP Server - <br/>
<h3>
 •	The next step is to install the DHCP server. We have already done this on via Powershell before installing the server but this is another method. <br/>
•	26:38 <br/>
•	Add the DHCP server through Manage / Add Roles / DHCP Server <br/>
•	After installation, manage Tools / DHCP to setup the Scope / IPv4 / New Scope (to assign available IP address range) <br/>
o	Name: 172.16.0.100-200<br/>
o	Router (Default Gateway): The domain controller is using its own NIC to provide internet access and will function as the Default Gateway 172.16.0.1 and finish the configuration <br/>
o	When complete, go back to the DHCP, right click to Authorize and Refresh <br/>
<h2> Allow internet access (not normally done in a production server) <br/>
<h3>
 •	Server Manager / Local Server / Internet Explorer Enhanced Security Configuration / turn off (Lab use only) <br/>
 <h2> Use Powershell script to create users <br/>
<h3>
  •	Open IE and download [script](https://github.com/joshmadakor1/AD_PS/archive/master.zip) / save to desktop 

 Powershell ISE Run as Admin / Open saved script folder “1_CREATE_USERS.ps1” <br/>

 Server isn’t allowed to run scripts by default so you must enable Run Script “Set-ExceutionPolicy Unrestricted” <br/>

 Change directory to file location “cd C:\users\a-kpierce\desktop\AD_PS-master” <br/>
	
 Click Play to create user accounts <br/>
	
 47:05 <br/>
	
 Somehow broke windows 10 vm 1 <br/>
	
 Labeled windows 10 vm 2: CLIENT1 domain: mydomain.com <br/>
## Creating a Windows 10 VM: CLIENT1 <br/>

 
 •	Installed Windows Media Creation Tool using an admin account. <br/
                                                                     
•	Created a VM in Virtualbox and assigned NIC of Internal: intent <br/>
•	Booted and installed Windows from Window10.iso <br/>
 
•	Ipconfig shows: Ethernet – 192.168.3.8 | 255.255.255.0 | no default gateway and Ethernet 2 – 169.254.254.253 | 255.255.0.0 | no default gateway <br/>
 
•	After installing guest additions and a restart, ipconfig shows:
o	Ethernet 1 - 192.168.3.8 | 255.255.255.0 | no default gateway
 
o	Ethernet 2 – “mydomain.com” IPv4 172.16.0.100 | 255.255.255.0 | gateway 172.16.0.1 (WOOHOO!)
 
•	Ping google.com works: indicates network and DNS are functional <br/>
 
•	Tracert 9.9.9.9 works: indicating network works and DC are functional<br/>
## Joining the domain with CLIENT1 <br/>
 

 •	Right click on start / Settings / Rename this PC (Advanced) / Change / CLIENT1 and mydomain.com <br/>
•	Authenticate with username and pw: kpierce | Password1  <br/>
 
•	Was able to log in and authenticate to the domain. Restart. <br/>
 
•	CLIENT1 lease shows up in the DC DHCP IPv4: 172.16.0.100 | CLIENT1.mydomain.com <br/>
•	Was able to log in to CLIENT1 as user: kpierce <br/>
 
•	DC / Active Directory Users and Computers / Computers shows CLIENT1 indicating that the computer is a has logged in and is a member of the domain. <br/>
•	Signed out of CLIENT1. Was able to log in as user: abargo | Password1<br/>
 
 
# SUCCESS!
 
# PHASE 3: Re-build AD DS lab in VMware Workstation 17 <br/>

 - Named VM: DC 
 - Configure 3 NICs: 
  - Ethernet0
   - MAC: 00-50-56-2B-13-5B
   - IPv4: Internet
   - Static IP: 
  - Ethernet1
   - MAC: 00-50-56-3D-F2-D7
  - Ethernet2
   - MAC: 00-50-56-34-F3-D7
 - After boot, press any key to run .ISO and install Server  2019
 - Assign Administrator Password: Password1
 - VMware has a button for "CTR + ALT + DEL"
 - Install VMware Tools for better display and mouse control | restart
 - Check Network Adapter Options to confirm installed NICs 
 - Could also use Control Panel | Network and Internet | Network Connections 
 - Need to identify NICs. Shutdown VM and review settings for Ethernet0, Ethernet1, and Ethernet2.
 - DC VM SETTINGS: 
  - NETWORK ADAPTER 1 - NAT
 MAC: 00:50:56:2B:13:5B
  - NETWORK ADAPTER 2 - Custom: Specific Virtual Network - VMnet0
 MAC: 00:50:56:3D:F2:D7
  - NETWORK ADAPTER 3 - Bridged: Connected directly to the physical network | Replicate physical network connection state
 MAC: 00:50:56:34:F3:D7
 
 Based on these findings, 
 
 Network Adapter 3: Bridged is assigned to Ethernet2
 
 Network Adapter 2: Custom is assigned to Ethernet1
 
 Network Adapter 1: NAT is assigned to Ethernet0 
 
 Rename adapters as follows:
 
 Network Adapter 3/Ethernet 2 is labeled  `X_INTERNAL1_X`
 
 Network Adapter 2/Ethernet1 is labeled `X_INTERNAL0_X`
 
 Network Adapter 1/Ethernet0 is labeled `_INTERNET_`
 
 
 Network Adapter 3/Ethernet 2 /`X_INTERNAL1_X`/00:50:56:34:F3:D7/Bridged should be redundant and needs to be disabled.
 
 Shutdown VM | Disable Network Adapter 3 | Uncheck "Connect at Power On" | Restart VM
 
 Looks like both _INTERNET_ and X_INTERNAL0_X NICs are able to connect to the internet. 
 
 `ipconfig /all` shows an IP address and internet connection
 
 `tracert 8.8.8.8` connects
 
 `tracert www.google.com` connects. DNS is functioning.
 
 Disable _INTERNET_ to test connection of X_INTERNAL0_X
 
 `ipconfig /all` shows APIPA address with a mask of 255.255.0.0 and no default gateway
 `ping 8.8.8.8` failure (expected)
 `tracert 8.8.8.8` failure
 
 Confirmed both _INTERNET_ and X_INTERNAL0_X are functioning correctly | Re-enable _INTERNET_ after testing
 
 
 
 
 
 
 
 
 
 
 
 
 

 

 
 
 
 
  




