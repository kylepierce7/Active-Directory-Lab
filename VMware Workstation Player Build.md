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
 
 jbob's password is incorrect. Log in as Admin and check/preform password reset. (NOT DONE YET)
 
 
 - Update Windows Server 20230417
 
 - Login as Admin and review jbob's account password. Administrative Tools / Active Directory Users and Computers / Enable Advanced features //
- Find Objects in Domain Services /  Entire domain / jbob / User found / Right click on user / Reset Password: Password2 (must change at next login) 
 
 - Make test disabled user account. Users and Computers / Users / Add User / User: Frank Smith / fsmith / Password1 / Status: disabled
 
 - Confirm guest accounts is disabled. 
 
 - Create a new Group for disabled accounts / Users / Disabled Accounts / Properties / Members / Add fsmith / Add Group or Username / Add Disabled Accounts to create a group / Deny All
- Restart system and confirm fsmith account / "Account has been disabled. See your system admin"
- Confirm jbob's password reset / System asks for a password update/ Password history doesn't allow Password1 for the rest / Password: Password3
- jbob's login presents with "The Sign-in method you're trying to use isn't allowed. Contact your network administrator."
- Login as Admin / Check Local Group Policy Editor `gpedit.msc` / Windows Settings / Security Settings / Local Policies / User Rights Assignments / Allow Log in locally / Properties / Denyed /




 
 
 
 ## Configure Remote Access and Network Address Translation for DC
 - Install Remote Access Server (RAS) and NAT. This will allow CLIETN1 to access the internet through the DC.
 - Server Manager / Manage / Add Roles and Features / Remote Access / Select "Direct Access and VPN", "Routing" & "Web Application Proxy"
- Tools / Routing and Remote Access / DC Local / Configure and enable / Configuration: NAT / Selected `_INTERNET_` ad internet interface / Network selection, selected `X_INTERNAL0_X`
 
 ## Configure DHCP server
 - Manage / Add roles and Features / DHCP server / Install
 - After install, Tools / DHCP / Setup the IPv4 Scope / New Scope (to assign the available IP address range)
 -IP address range:
  - Name: `172.16.0.100-200
  - Subnetmask: 255.255.255.0
  -Router ( Default Gateway): 172.16.0.1 Add (The DC is using its own NIC to provide internet access and will function as the Default Gateway `172.16.0.1` )
 - Finish the configuration / When complete go back to the DHCP and Right Click to Authorize and Refresh
 
 
 ## Install Windows 10 Pro
 - Windows Media Creation Tool / Create installation media (I already had an .ISO)
 
-Name machine: CLIENT1 / Password: Password1 / pet: Password1 / City: Password1 / City where parents met: Password1
- Install VMware Tools 
- was able to log in as CLIENT1
- Shutdown
- Change VMware Settings / NIC: Custom - VMnet0
- Restart / attempt to login to domain / Not available
- VM has 3 NICs:
1. Custom: VMnet0
2. NAT
3. Host-only
 
# Joining the Domain with CLIENT1
- Right click on Start / Settings / Rename this PC (Advanced) / Change / Computer Name: CLIENT1 / Domain: mydomain.com
- Authenticate: Was able to authenticate with `a-kpierce` and `Password1`
- Restart
 - Was able to login to Domain as `jbob` `Password3`
 - Go to Active Directory DC and change `jbob`'s password back to `Password1`
 - Active Directory Users and Computers / Find Objects `jbob` /  Right click to change password / `Password1`
 - Go back to CLIENT1 to confirm / Can login in to `jbob` with `Password1`
 
 •	SUCCESS!
 
## Checking other user account permissions 
- fmith: Couldn't login. Login to the DC and retry. "We can't sign you in with this credential because your domain isn't available..."
- jbob: valid login
- Go to the DC and enable fsmith's account and retry. Active Directory Users and Computers / Find Users and Computers / fsmith / Right click to enable / 
- fsmith still can't login. Check AD user settings at DC. Active Directory Users and Computers / Frank Smith Properties / Unlock Account / Did not work / RESTART CLIENT1
- fsmith can login to CLIENT1
	
- SUCCESS!
	
	## Use a Powershell script to create users
	- SOURCE: Youtube.com | Josh Madakor | "How to setup a basic home lab running active directory" | https://www.youtube.com/watch?v=MHsI8hJmggI&t=1681s
	
	

$PASSWORD_FOR_USERS   = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
	
	
 
Saved the above as `1_CREATE_USERS.ps1` / Saved to DC Desktop
	
Entered a txt file of sample user names as `names.txt` / Saved to DC Desktop
	

	Powershell ISE Run as Admin / Enable scripts on server `Set-ExecutionPolicy Unrestricted` / Right click on file to find Path / `C:\Users\a-kpierce\Desktop` | Need to elivate privlege so run as `".\1_CREATE_USERS.ps1"Navigate to `1_CREATE_USERS.ps1` folder is and RUN 
 
	** code error in `New_AdUser -AccountPassword $pasword` ** 
	
	** FIX TO: `New_AdUser -AccountPassword $password` | EDIT FILE and RETRY 
	
	** Same error. Troubleshoot script.
	
	
	

 
 
 
 
 
 
