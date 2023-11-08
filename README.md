# Nmap Network Enumeration

In this project...
## Host Enumeration

To conduct an internal penetration test it is beneficial to first see which systems are online. ICMP echo requests are effective in discovering these hosts. 
### Host Discovery
#### Scanning a Network Range

I first use an IP range to scan for open hosts: 

![](Images/Pasted%20image%2020231108141021.png)

These options are used to: 
- `-sn` = skip port scanning 
- `-oA hosts` = store results in formats starting with hosts

Then I create a list of IP addresses with the following IPs:

![](Images/Pasted%20image%2020231108141343.png)

With the list I can then perform an identical scan but with the predefined list rather than a provided range using the `-il hosts.lst` option:

![](Images/Pasted%20image%2020231108141359.png)

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

#### Scanning Top Ports
Now with knowledge of the open hosts I can begin to scan for ports, services, and operating system information. 

First I scan a target host with Nmap's list of most frequently used ports:

![](Images/Pasted%20image%2020231107151007.png)

With `--top-ports=10` I specify to grab the top ten most used ports from Nmap's list. 

To get a more detailed look at the closed FTP port I use a SYN scan on the specific port to verify the RST packet is received from the host: 

![](Images/Pasted%20image%2020231107152036.png)

Using `--packet-trace -Pn -n --disable-arp-ping` disables ICMP echo, DNS, and ARP pings while also showing all the packets sent and received to verify the response from the host. In this example I can see that the host responded (in the RCVD line) with a TCP packet containing the flags RA, meaning it responded with the RST and ACK packets signaling that the port was indeed closed. 

#### TCP Connect Scan

To get more accurate results on if a port is open or not, I use `-sT` to enable a TCP connection scan on HTTP/HTTPS ports: 

![](Images/Pasted%20image%2020231107161001.png)

#### Filtered Ports

Some ports are displayed as filtered and doing a packet trace on these ports can help determine if the host dropped or rejected the packets I have sent it. 

Doing a packet trace on the filtered port 445 (with the default max retries of 1) shows that the overall scan time took much longer than usual at 2.11 seconds, meaning that the packet was likely dropped: 

![](Images/Pasted%20image%2020231107163744.png)

If the host's firewall had rejected the packets then the scan time would have been much shorter, for example 0.05 seconds. It would also be likely that I would receive an ICMP reply with type 3 and error code 3 showing that the host would be unreachable. 

#### UDP Scans

Checking for UDP ports is trickier because UDP packets do not require responses like TCP connections do. Any open UDP ports that send responses back are because the application is configured to do so. 

Here I run a scan for open UDP ports and do a packet trace on one that is listed as open: 

![](Images/Pasted%20image%2020231107170905.png)

#### Scanning for Versions

Another useful scan on specific ports is to check for their versions, service names, and other service info:

![](Images/Pasted%20image%2020231107174044.png)

### Saving the Results

Using the option `-oA` I save the results of these scans to files that are readable, grepable, and also in XML format:

![](Images/Pasted%20image%2020231107191315.png)

Also, I can convert the XML output file to a report-style HTML file using the tool **xsltproc**: 

`xsltproc target.xml -o target.html` 

![](Images/Pasted%20image%2020231107191628.png)

### Service Enumeration

Scanning for services gives information on the types of applications being used, their versions, and the known vulnerabilities that can be exploited for those versions. 

Using a quick scan of all ports and their services with `-p- -sV` I can get all TCP ports with their service and version: 

![](Images/Pasted%20image%2020231108123618.png)
#### Banner Grabbing 

Sometimes Nmap doesn't know how to handle all the info that it receives since banners aren't required to be given immediately. One way to circumvent this is connecting to the service directly with netcat and capturing the traffic with tcpdump: 

![](Images/Pasted%20image%2020231108140024.png)
### Nmap Scripting Engine



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
- unfiltered = tcp-ack scan shows port accessible but cant tell if open/closed
- open|filtered = dont get response from specific port; firewall or packet filter 
- closed|filtered = IP ID idle scans; impossible to determine if port is closed or filtered

ex: sudo nmap -sS localhost

default will scan top 1000 TCP ports with SYN 

SYN is only default when ran as root because of socket permissions required to create raw TCP packets 

TCP scan will default without sudo

defining ports: 
- individual = `-p 22, 25, 80`
- range = `-p 22-445`
- top ports = `--top-ports=10`

`-sT` = TCP connection scan 
- more stealth since it does not leave any unfinished connections or unsent packets 
- won't disturb services but still able to map network 
- most accurate way of determining state of port 
- evades firewalls that drop incoming packets but allow outgoing packets 
- slower because it waits on each response

UDP scans are sometimes forgotten to be filtered and also don't receive a response like TCP 

UDP is also much longer with timeout times

if a response is given from a UDP scan and the port is listed as "open" then it is because the app is configured to respond, otherwise there is no way to tell if the packet arrived or not

save results in 3 different formats: 
- normal output with .nmap `-oN`
- grepable output with .gnmap `-oG`
- XML output with .xml `-oX`

`-oA` will save in all 3 formats

version scanning 
- start with port scan `-p- -sV`
- can get stats in intervals `--stats-every=10s`
- can change verbosity to see when ports are detected `-v` `-vv`

banner grabbing
- tries to id service through banners
- if banners don't work then it uses signature detection but takes a lot longer

Sometimes nmap gets results that it doesn't know how to handle 

- `NSOCK INFO [0.4200s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 18 [10.129.2.28:25] (35 bytes): 220 inlane ESMTP Postfix (Ubuntu)..`

in this example, the server sends banner after TCP handshake with PSH flag at network level

some services don't always provide the banner info 

another way to get banners is to manually connect to the server using nc, grab banner, and intercept traffic using tcpdump

ex: 
- start packet capture `sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28`
- connect with netcat `nc -nv 10.129.2.28 25`

in the tcpdump logs you can verify TCP handshake 
- Flags [S] = SYN 
- Flags [S.] = SYN-ACK
- Flags [.] = ACK 
- Flags [P.] = PSH and ACK meaning target server is sending us data and that all required data is sent
- Flags [.] = ACK to confirm receipt of data