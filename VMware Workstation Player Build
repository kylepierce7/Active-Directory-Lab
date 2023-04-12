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
