
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

NSE = nmap scripting engine 

scripts use LUA 
- auth = auth creds
- broadcast = host discovery by broadcast 
- brute = try to log in with brute force
- default = -sC option
- discovery = find services
- dos = check for DoS vulns 
- exploit = exploit vulns for port 
- external = external services for further processing 
- fuzzer = id vulns and unexpected packet handling by sending different fields
- intrusive = intrusive scripts that could impact system
- malware = checks if malware exists on system 
- safe = defensive scripts that are not intrusive
- version = extension for service detection 
- vuln = id of specific vulns

performance 
- how fast = `-T<0-5>`
- frequency = `--min-parallelism <number>` 
- timeouts = `--max-rtt-timeout <time>`
- how many packets should be sent simultaneously `--min-rate <number>`
- number of retries `--max-retries <number>`

RTT = round trip time = how long it will take to receive a response from port after nmap sends packet

generally --min-rtt-timeout = 100ms

Timing and aggressiveness
- `-T 0` / `-T paranoid`
- `-T 1` / `-T sneaky`
- `-T 2` / `-T polite`
- `-T 3` / `-T normal`
- `-T 4` / `-T aggressive`
- `-T 5` / `-T insane`

firewall = monitor net traffic and handles connections based on rules
IDS = reports detected attacks based on network scans
IPS = block after detection

dropped packets from firewall = ignored and no response
rejected = return with RST flag 

RST packet contains different types of ICMP error codes 
- net unreachable 
- net prohibited 
- host unreachable 
- host prohibited 
- port unreachable 
- proto unreachable 

TCP ACK scan `-sA` is much harder to filter compared to `-sS` or `-sT` 

when port is open or closed, responds to ACK with RST flag 

most incoming connection attempts, with SYN flag for example, are usually filtered

ACK flags are typically passed because the firewall can't determine if connection was established from inside or outside network 

IDS/IPS detection is much more difficult because they are passive and examine all connections between hosts

it is recommended to use several VPS with different IPs to determine if IDS/IPS are active on host because IPs will typically instantly get blocked if detected

can intentionally do aggressive scans to check for a security response 

decoy scanning can be used when specific subnets from different regions are blocked or if the IPS should block 

decoy = nmap generates random IP addresses into IP header to disguise origin of packet 

`-D RND:5` = generate 5 random IP addresses

the real IP address will be placed in between the randomly generated IPs 

decoys must be alive because then the service on the target may be unreachable due to SYN flooding security mechanisms 

can specify VPS servers IPs and use them in combo with `IP ID` manipulation in IP header

specify source IP address with `-S`

mostly UDP port 53 for DNS 

TCP port 53 used to be only for zone transfers but changing now with IPv6

can specify DNS server with `--dns-server <ns>,<ns>` 

if in a DMZ this will be very valuable 

company's DNS servers are trusted more than those from internet

could use org's DNS servers to interact with hosts on internal network 

can use TCP port 53 as source port with `--source-port`, if admin uses firewall to control this port and doesn't filter properly then TCP packets will be trusted