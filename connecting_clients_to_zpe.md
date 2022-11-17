# Creating VLAN

   1 - Go to Network::Connections and click on "Add"
   
       Name: VLAN90             IPv4 Mode: Static
       
       Type: VLAN                    IP Address: 172.16.160.99
       
       Interface: backplane0         Bitmask: 24
       
                                     Gateway IP: 172.16.160.1
       
       VLAN ID: 90
       
   <img width="1777" alt="Screen Shot 2022-10-04 at 16 20 09" src="https://user-images.githubusercontent.com/103506681/193829932-13167a93-1546-4af6-87ed-4623b4596aab.png">

  2 - Assign an static IP address to BACKPLANE0
  
      IPv4 Mode: Static
                 
        IP Address: 172.16.160.2
        
        BitMask: 24
        
        Gateway IP: 172.16.160.1
        
  3 - Create VLAN for switches
  
  Go to Network::Switch::VLAN and click on "Add"
  
      VLAN: 90
      
      Select Untaggaed Ports: backplane0
                              netS1
                              netS2
                              netS3
                              netS4
                              
  4 - Create Static Route for BACKPLANE0
      
  Go to Network::Static Routes and click on "Add"
  
      Index: route1               Destination IP: 0.0.0.0
      
      Connection: BACKPLANE0      Destination Bitmask: 0
      
      Type: ipv4
  
  <img width="1788" alt="Screen Shot 2022-10-04 at 16 24 53" src="https://user-images.githubusercontent.com/103506681/193831062-4c2ee2a4-fc8f-47fb-ae29-6f28d3b63e4f.png">

  5 - Create DHCP Server
  
  Go to Network::DHCP Server and click on "Add"
  
      Protocol: DHCP4
          
          Subnet: 172.16.160.0
          
          Netmask: 255.255.255.0
          
          Domain Name Servers: 8.8.8.8
          
          Router IP: 172.16.160.99
          
          Lease Time (s): 86400
          
   <img width="892" alt="Screen Shot 2022-10-04 at 16 27 06" src="https://user-images.githubusercontent.com/103506681/193831541-8deaefe5-9b58-439b-9ad1-9e51c8d54816.png">
    
  Go to "Network Range" under DHCP Server and click on "Add"
  
      IP Address Start: 172.16.160.2
      
      IP Address End: 172.16.160.98
      
  6 - Create NAT Postrouting rule
  
  Go to Security::NAT::POSTROUTING:IPv4 and click on "Add"
  
      Target: MASQUERADE
      
      Rule Number: 0
      
      Source IP/Mask: 172.16.0.0/255.255.0.0
      
      Destination IP/Mask: 0.0.0.0/0.0.0.0
      
      Output Interface: Any
      
  <img width="1688" alt="Screen Shot 2022-10-04 at 16 30 44" src="https://user-images.githubusercontent.com/103506681/193832452-cd8a71a2-3186-4b52-82ca-668dba9b526c.png">

  
  
      
  
      
  
      
