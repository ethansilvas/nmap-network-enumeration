# Nmap Network Enumeration

In this project...
## Host Enumeration

To conduct an internal penetration test it is beneficial to first see which systems are online. ICMP echo requests are effective in discovering these hosts. 

### Host Discovery
#### Scanning a Network Range

I first use an IP range to scan for open hosts: 

![](Images/Pasted%20image%2020231106190952.png)

These options are used to: 
- `-sn` = skip port scanning 
- `-oA tnet` = store results in formats starting with tnet

Then I create a list of IP addresses with the following IPs:

![](Images/Pasted%20image%2020231106192214.png)

With the list I can then perform an identical scan but with the predefined list rather than a provided range using the `-il hosts.lst` option:

![](Images/Pasted%20image%2020231106192249.png)

With both of these searches the outputs are stored in the specified files with name tnet: 

![](Images/Pasted%20image%2020231106192510.png)

Some other ways I could do this either use specific IP addresses or specify the range in the octet: 

`sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20 | grep for | cut -d" " -f5`

`sudo nmap -sn -oA tnet 10.129.2.18-20 | grep for | cut -d" " -f5`

#### Scan Single IP Addresses

Before performing port, service, OS, etc. scanning on an IP, it is good to check to see if the host is alive and responsive. 

I use the same scan before on the target IP addresses to see which ones are up: 

![](Images/Pasted%20image%2020231106193802.png)

I can then use the following options to get more info on why the host is described as being up: 
- `-PE` = send ICMP echo requests 
- `--reason` = output the type of reply that was received from the host

![](Images/Pasted%20image%2020231106194509.png)
### Host and Port Scanning



## Bypass Security Measures

## Notes 

### Enumeration

enumeration = identify all ways you could attack a target, rather than gaining access to target

vital to understand the services to look for and the opportunities they provide 

what syntax they use for communication + interaction with different services 

gaining access to target system can be narrowed down to: 
- functions and resources that help us interact with target 
- info that provides us with more important info to access our target
### Introduction to Nmap

Nmap = network mapper
- scan networks 
- id which hosts are available on network using raw packets and services/apps 
- names and versions 
- OS detection and versions 
- scanning for misconfigs in packet filters, firewalls, or IDS

use cases: 
- audit security aspects of networks 
- pen test
- check for misconfigs
- types of possible connections
- response analysis 
- port scanning 
- vuln assessment 

types of scans: 
- host discovery 
- port scanning 
- service enumeration and detection 
- OS detection 
- script interaction

basic syntax: `nmap <scan type> <options> <target>`

default to do -sS 
- syn packet 
- never completes handshake 
- if syn/ack packet returned, then port is open
- RST = closed 
- no packet = filtered 

ex: sudo nmap -sS localhost