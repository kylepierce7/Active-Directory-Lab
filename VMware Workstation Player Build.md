# Duplicate & Re-build AD DS lab in VMware Workstation 17 <br/>

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
 
 Configure IP addressing: Network Settings / Change Adapter Options  / Properties / IPv4 Properties
 
 `X_INTERNAL0_x` 
 - 172.16.0.1
 - 255.255.255.0
 - Gateway: <empty>
 - DNS: 127.0.0.1
 
## Change Computer name / Settings / Home / About / System Info / Change Settings / Change / Changed to "DC" / Restart system
 
 Active Directory will act as its own DNS server after it's installed. That's why the loopback address (127.0.0.1) was used. 
 
 ## Active Directory Configuration
 - Install Active Directory Domain Services (AD DS) and create a domain 
  - Server Manager / Add Roles and Features / Active Directory Domain Services 
  - Promote server to Domain Controller: Active Directory Domain Services Configuration Wizard / Add new forest / add domain name: `mydomain.com`
 
 Password: Password1
 
 **NOTE** Active Directory Domain Services Configuration Wizard shows this error:
 
 "A delegation for this DNS server cannot be created because the authoritative parent zone cannot be found or does not run Windows DNS server. If you are integrating with an existing DNS infrastructure, you should manually creat a delegation to this DNS server in the parent zone to ensure reliable name resolution from outside the domain "mydomain.com". Otherwise, no action is required."
 
 NetBIOS domain name: MYDOMAIN
 
 After restart, was able to login as Admin to the MYDOMAIN.COM domain!
 
 ## 
 
 # Creating Admin Accounts
 
 - Start / Administrative Tools / Active Directory Users and Computers
 - `mydomain.com` create a new Organizational Unit to store the new Admin account. `_ADMINS`
 In `_ADMINS`, create a new user `a-kpierce` (`a-` represents an Admin account). Password: Password1
 Edit User Properties / Member Of / Add / `Domain Admins` 
 Sign out and you can now log in under the `Other User` with your account name and password.
 
 # Creating User Accounts
 - Start / Administrative Tools / Active Directory Users and Computers / Users / New User
 - Make new user: Jim Bob/ jbob / Password: Password2 / User must change on login / Will change to: Password1
 - Sign out / Log in as `jbob` / System prompts to change password / jbob
 
 
 NEXT STEP:
 - jbob's password is incorrect. Log in as Admin and check/preform password reset. (NOT DONE YET)
 
 
 
 
 
