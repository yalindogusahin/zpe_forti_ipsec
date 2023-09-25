## Introduction

This guide provides step-by-step instructions for configuring an IPsec tunnel between your ZPE device and a FortiGate firewall, ensuring secure communication between the two devices. Additionally, this repository includes an [Ansible playbook](https://www.ansible.com/) that simplifies the process of adding an IKE profile to Nodegrid for establishing the IPsec tunnel.

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Forti Configuration](#forti-configuration)
  - [Create IPsec Tunnel](#create-ipsec-tunnel)
  - [Tunnel Configurations](#tunnel-configurations)
  - [Create Static Route](#create-static-route)
  - [Create IPv4 Policy](#4a-create-ipv4-policy)
- [Nodegrid Configuration](#nodegrid-configuration)
  - [Create IKE Profile](#1-create-ike-profile)
  - [Creating the Tunnel](#2-creating-the-tunnel)
- [Testing the Tunnel](#testing-the-tunnel)
- [Useful Documentations](#useful-documentations)
- [Conclusion](#conclusion)
  
## Prerequisites

Before you begin, make sure you have the following:

- Access to the FortiGate firewall with administrative privileges.
- Access to the ZPE device with administrative privileges.
- Knowledge of your ZPE device's public IP address (ZPE_PUBLIC_IP_ADDR).
- Knowledge of your ISP interface name (ISP_INTERFACE).
- A secure pre-shared key (password) for authentication.

> **Note:** This guide assumes you have the necessary access and privileges on both devices.

## Forti Configuration

### Create IPsec Tunnel

Go Under VPN::IPsec Wizard
 
    Name: $Spesify
    
    Template Type: Custom
    
![Screen Shot 2022-09-30 at 16 16 18](https://user-images.githubusercontent.com/103506681/193278033-e13c3c84-477c-4aa8-9e29-7e7443134e87.png)

2 - Tunnel Configurations
    
    IP Version = IPv4
    
    Remote Gateway = Static IP Address
    
    IP Address = $ZPE_PUBLIC_IP_ADDR
    
    Interface = $ISP_INTERFACE
    
    Local Gateway = Disabled
    
    Mode Config = Uncheck
    
    Nat Traversal = Enable (If your firewall behind NAT Enable it)
    
    Keepalive Frequency = 10
    
    Dead Peer Detection = On Demand
    
    --------------------------------------
    
    Authentication Method = Pre-shared Key
    
    Pre-shared Key = $password
    
    IKE Version = 2
    
    --------------------------------------
    
    Phase 1 Proposal
    
    Encryption = AES256     Authentication = SHA256
    
    Diffie-Hellman Group = 21 (uncheck all others)
    
    Key Lifetime (seconds) = 28800
    
    Local ID = forti
    
    --------------------------------------
    
    Phase 2 Selectors
    
    Local Address = $LOCAL_SUBNET
    
    Remote Address = $REMOTE_SUBNET
    
    Encryption = AES256     Authentication = SHA256
    
    Enable Replay Detection = Check
    
    Enable Perfect Forward Secrecy (PFS) = Check
    
    Diffie-Hellman Group = 21
    
    Local Port = All (Check)
    
    Remote Port = All (Check)
    
    Protocol = All (Check)
    
    Auto-negotiate = Check
    
    Autokey Keep Alive = Check
    
    Key Lifetime = Seconds
    
    Seconds = 3600

![Screen Shot 2022-09-30 at 16 22 45](https://user-images.githubusercontent.com/103506681/193281671-7cc9c61e-38a9-4fec-8891-549efb8f7956.png)

<img width="642" alt="Screen Shot 2022-09-30 at 16 36 37" src="https://user-images.githubusercontent.com/103506681/193281739-f1508862-d737-4911-a2a5-63fcd58e4645.png">

3 - Create Static Route
    
Go to Network::Static Routes and click on "Create New"
    
    Dynamic Gateway = Check
    
    Destination = Subnet (172.16.160.0/24 in my case)
    
    Interface = ZPE_IPsec
    
    Administrative Distance = 10
    
    Status = Enabled
    
<img width="645" alt="Screen Shot 2022-09-30 at 16 40 42" src="https://user-images.githubusercontent.com/103506681/193282518-c622b075-40f2-40fd-8b91-6c2d0669f039.png">

4 - 
a)  Create IPv4 Policy

Go under Policy & Objects::IPv4 Policy and click on "Create New"

    Name = Local > VPN
    
    Incoming Interface = $LOCAL_SUBNET_INTERFACE (internal1 in my case)
    
    Outgoing Interface = ZPE_IPsec
    
    Source = all
    
    Destination = all
    
    Schedule = all
    
    Service = all
    
    Action = ACCEPT
    
    ---------------
    
    NAT = Disabled
    
    IP Pool Configuration = Use Outgoing Interface Adress
    
    Preserve Source Port = Disabled
    
    ----
    
    AntiVirus = Disabled
    
    Web Filter = Disabled
    
    DNS Filter = Disabled
    
    Application Control = Disabled
    
    IPS = Disabled
    
    SSL Inspection = Disabled
    
    Log Allowed Traffic = Enabled (Security Events)
    
    Enable this Policy
    
<img width="910" alt="Screen Shot 2022-10-04 at 16 15 32" src="https://user-images.githubusercontent.com/103506681/193828898-bd59b37f-88be-4fb8-9c3f-074e72b33575.png">

    
    b) Clone Reverse of Policy that we created for Local > VPN and name it VPN > Local
    
<img width="345" alt="Screen Shot 2022-09-30 at 16 46 47" src="https://user-images.githubusercontent.com/103506681/193284093-4396a0e1-902d-4c00-8688-bdaa808802a5.png">

# Nodegrid Configuration

    1 - Create IKE Profile
    
Go to Network::VPN::IPsec::IKE Profile and click on "Add"
 
        Profile Name = Forti
        
        IKE Version = IKEv2
        
        --------------------------------
        
        Phase 1
        
        Encryption = AES256
        
        Authentication = SHA256
        
        Diffie-Hellman Group = Group 21
        
        Lifetime (s) = 28800
        
        --------------------------------
        
        Phase 2
        
        Authentication Protocol = ESP
        
        Encryption = AES256
        
        Authentication = SHA256
        
        PFS Group = Group 21
        
        Lifetime = 3600
        
        --------------------------------
        
        Advanced Settings
        
        Enable Dead Peer Detection = Check
        
        Number of Retries = 5
        
        Interval (s) = 10
        
        Action = restart
        
        MTU = 
        
        Custom Parameters = 
        
    2 - Creating the Tunnel
    
Go to Network::VPN::IPsec::Tunnel and click on "Add"

        Name = zpe                                  Authentication Method = Pre-Shared Key
        
        Initiate Tunnel = Start                     Secret = $YOURSECRET
        
        IKE Profile = Forti
        
        ------------------------------------------------------------------------------------------------
        
        Local:                                       Remote: 
        
        Left ID = @zpe                               Right ID = @forti
        
        Left Address = %wwan0                        Right Address = Forti Public IP Address
        
        Left Source IP Address = 172.16.160.1        Right Source IP Address = 192.168.254.3
        
        Left Subnet = 172.16.160.0/255.255.255.0     Right Subnet = 192.168.230.0/255.255.255.0
        
        ------------------------------------------------------------------------------------------------
        
        Virtual Tunnel Interface
        
        Enable Virtual Tunnel Interface = Enable
        
        Mark = -1
        
        Address = 172.16.160.1/24
        
        Interface = vti-forti
        
        Automatically create VTI routes = check
        
        Share VTI with other connections = check
        
# Test the tunnel

  1 - Connect the nodegrid console
  
      shell sudo su -
      
      ping 192.168.230.99 -I vti-forti
      
![Screen Shot 2022-10-04 at 14 28 56](https://user-images.githubusercontent.com/103506681/193808818-38b77560-53ab-48a9-b915-a36f1d0a203e.png)

## Conclusion

Configuring an IPsec tunnel between your ZPE device and a FortiGate firewall is a crucial step in ensuring secure and reliable communication between your network endpoints. This guide has provided you with a comprehensive walkthrough of the configuration process, covering everything from initial setup to testing.

By following these steps, you've established a robust security mechanism that allows your devices to exchange data over an encrypted connection. The combination of ZPE's capabilities and the FortiGate firewall's robust security features ensures the confidentiality and integrity of your data.

Remember that maintaining the security of your IPsec tunnel is an ongoing process. Regularly monitor and update your configurations to adapt to evolving security threats and network requirements. Troubleshooting resources are also available to help you address any issues that may arise during the operation of your IPsec tunnel.

With a well-configured IPsec tunnel in place, you can confidently use your network resources while knowing that your data remains protected against unauthorized access and interception. This knowledge empowers you to leverage secure communication for your business operations, data transfers, and more.

        
# Usefull documentations

  1 - Troubleshoot on Forti Side
      
      https://community.fortinet.com/t5/FortiGate/Troubleshooting-Tip-IPsec-VPNs-tunnels/ta-p/195955
      
      https://community.fortinet.com/t5/FortiGate/Troubleshooting-Tip-Troubleshooting-IPsec-Site-to-Site-Tunnel/ta-p/195672
      
      https://docs.fortinet.com/document/fortigate/6.2.10/cookbook/044240/ipsec-related-diagnose-command
      
      https://www.youtube.com/watch?v=91GznQt2kzg
      
  2 - Troubleshoot on Nodegrid
  
      https://support.zpesystems.com/portal/en/kb/articles/how-to-troubleshoot-ipsec-issues
      
 ![image](https://user-images.githubusercontent.com/103506681/194721397-f97460b8-de2b-42a9-9061-56d350c92e02.png)


        


    
    
    
    
    
    
    
    
    
    
