# Cisco IOS - Life of a Packet: DNS Lookup and ARP Behavior
In this lab, I have built and configured from scratch a simple network in Cisco Packet Tracer to demonstrate how the Domain Name System and Address Resolution Protocol work together across different subnets.

## Network Topology
<img width="1560" height="563" alt="imagen" src="https://github.com/user-attachments/assets/52bc1270-cf45-417e-8b81-1476ce94dd93" />

The topology above shows an interconnected network divided into three LAN subnets plus a point-to-point (P2P) link connecting the two routers.

## Objective

My goal is to illustrate the complete **life of a packet** when **Host A** initiates communication with the web server's FQDN (webserver.com). This project details:
- How DNS resolution maps hostnames to IP addresses
- How ARP resolves Next-Hop MAC addresses dynamically
- How Layer 2 framing changes at every hop while Layer 3 IP headers remain intact end-to-end

## Device Configurations

Below you can find the CLI configuration snippet for each device:

### Router R1
```
Current configuration : 802 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R1
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX15243954-
!
!
!
!
!
!
!
!
!
!
!
ip name-server 10.10.100.10
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0
 ip address 10.10.10.1 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 ip address 10.10.100.1 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 ip address 10.10.11.1 255.255.255.252
 duplex auto
 speed auto
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
ip route 10.10.12.0 255.255.255.0 10.10.11.2 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```

### Router R2
```
Current configuration : 761 bytes
!
version 15.1
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname R2
!
!
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX152413YH-
!
!
!
!
!
!
!
!
!
!
!
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0
 ip address 10.10.11.2 255.255.255.252
 duplex auto
 speed auto
!
interface GigabitEthernet0/1
 ip address 10.10.12.1 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
ip route 10.10.10.0 255.255.255.0 10.10.11.1 
!
ip flow-export version 9
!
!
!
!
!
!
!
line con 0
!
line aux 0
!
line vty 0 4
 login
!
!
!
end
```

### DNS Server 
<img width="698" height="274" alt="imagen" src="https://github.com/user-attachments/assets/7ebb098e-ef49-4941-a66b-1bb6e8f5e190" />
<img width="692" height="383" alt="imagen" src="https://github.com/user-attachments/assets/e2b0f180-4d53-461e-b9d5-421c1765fe32" />

### Web Server
<img width="705" height="268" alt="imagen" src="https://github.com/user-attachments/assets/8aabb991-45a7-4833-9a72-c855ce7b02b8" />

TCP Ports 80 and 443 for web traffic are listening on the server

<img width="712" height="230" alt="imagen" src="https://github.com/user-attachments/assets/ecb2fdb0-ef99-4170-a91b-c24dd85cc4a6" />

## DNS & ARP requests 

- **Host A** which has the IP 10.10.10.10 belongs to the pink 10.10.10.0/24 subnet, and wants to send a packet to the FQDN `webserver.com` at 10.10.12.10, but initially it doesn't know the destination IP address, so it will hold the original packet and send a DNS request to its DNS server at 10.10.100.10 (the DNS server is already configured at this point)
- **Host A** knows that its own IP address is on a different subnet of the DNS server's IP address, and that's why the DNS request needs to be sent via router R1
- **Host A** will hold the DNS request and first send a broadcast ARP request to the switch SW1 looking for its default gateway MAC address

<img width="703" height="276" alt="imagen" src="https://github.com/user-attachments/assets/cfac433c-4419-40ab-ac38-80dcd93da02f" /> 

- In this point, the broadcast ARP request is received by the switch SW1
- SW1 then will add an entry in its MAC address table mapping Host A's MAC address 0007.EC96.2DE8 to port Fa0/1, so it knows that Host A is reachable through this port

<img width="321" height="107" alt="imagen" src="https://github.com/user-attachments/assets/43652338-237e-4254-8987-2d0e92b8ecb4" />

- SW1 will flood the broadcast traffic out all ports apart from the one it was received on (SW1 do this because FFFF.FFFF.FFF is the layer 2 broadcast address, it means "send the traffic everywhere"), in this case out only port Fa0/2
- The broadcast ARP request will hit router R1's interface 10.10.10.1, R1 will process the request and see it is for itself, so it will send  a unicast ARP reply back to host A

<img width="711" height="280" alt="imagen" src="https://github.com/user-attachments/assets/297501a3-4375-4221-97f2-1a8e914d0bcd" />

- Router R1 will also add an entry for host A mapping IP address 10.10.10.10 to MAC address 0007.EC96.2DE8 to its ARP cache

<img width="571" height="56" alt="imagen" src="https://github.com/user-attachments/assets/816510ab-1778-4908-a8ad-f00d785f1aed" />

- Switch SW1 will add another entry in its MAC address table mapping router R1's MAC address 00d0.97c7.8901 to port Fa0/2

<img width="327" height="120" alt="imagen" src="https://github.com/user-attachments/assets/735c749d-35a7-47a0-b07c-69a7fce4b2e7" /> 

- Then SW1 will send the unicast ARP reply out only port Fa0/1 which host A is plugged into because SW1 already has that entry in its MAC address table
- Finally host A recieves the unicast ARP reply and adds an entry for router R1 mapping IP address 10.10.10.1 to MAC address 00d0.97c7.8901 to its ARP cache. It will use this whenever it needs to send traffic (like the previous DNS request) to another IP subnet

<img width="418" height="162" alt="imagen" src="https://github.com/user-attachments/assets/6aed60da-2c56-413a-853a-181e60d85e6c" />

---

**Remember the DNS request host A was holding?** At this point host A can send it to de DNS server with the address 10.10.100.10 on the green subnet

<img width="920" height="658" alt="imagen" src="https://github.com/user-attachments/assets/58870ed6-fac1-47a6-b663-ea99213ddf01" />

- SW1 sees the destination MAC address and beacuse of its MAC address table knows it has to send the DNS request out only port Fa0/2 which router R1 is reachable
- Router R1 will recieve the DNS request packet and see that the destination IP address belongs to the green 10.10.100.0/24 subnet, which is actually directly connected to because it has an interface in that subnet, so it knows the destination IP address should be available out interface Gig0/1
- R1 doesn't know the MAC address of 10.10.100.10 so it will hold the DNS request packet and send a broadcast ARP request out of its interface Gig0/1

<img width="906" height="661" alt="imagen" src="https://github.com/user-attachments/assets/f000ca3c-72f7-40d0-b592-db6d5404fa75" />















