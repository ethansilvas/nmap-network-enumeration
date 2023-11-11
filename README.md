# Nmap Network Enumeration

In this project...
## Host Enumeration

Nmap provides many tools to discover responsive hosts in a provided IP range and then perform scans to learn valuable information about the host that could be used as intrusion methods. In this section I will go through some of the most important tools on an example target with the following steps: 

1. Discover any responsive hosts within a range of IP addresses
2. Perform port scanning on the responsive hosts
3. Save the results in different formats
4. Discover application services for the different ports found on the hosts
5. Use Nmap Scripting Engine to discover potential vulnerabilities based on the discovered services

After these scans have been completed, I will then test different performance optimization techniques and compare the results to more default scanning options. 
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

With both of these searches the outputs are stored in the specified files with name hosts: 

![](Images/Pasted%20image%2020231108141448.png)

Some other ways I could do this either use specific IP addresses or specify the range in the octet: 

`sudo nmap -sn -oA hosts 10.129.2.18 10.129.2.19 10.129.2.20 | grep for | cut -d" " -f5`

`sudo nmap -sn -oA hosts 10.129.2.18-20 | grep for | cut -d" " -f5`
#### Scan Single IP Addresses

Before performing port, service, OS, etc. scanning on an IP, it is good to check to see if the host is alive and responsive. 

I use the same scan before on the target IP addresses to see which ones are up: 

![](Images/Pasted%20image%2020231108141627.png)

I can then use the following options to get more info on why the host is described as being up: 
- `-PE` = send ICMP echo requests 
- `--reason` = output the type of reply that was received from the host

![](Images/Pasted%20image%2020231108141708.png)
### Host and Port Scanning

#### Scanning Top Ports
Now with knowledge of the open hosts I can begin to scan for ports, services, and operating system information. 

First I scan a target host with Nmap's list of most frequently used ports:

![](Images/Pasted%20image%2020231108141746.png)

With `--top-ports=10` I specify to grab the top ten most used ports from Nmap's list. 

To get a more detailed look at the closed FTP port I use a SYN scan on the specific port to verify the RST packet is received from the host: 

![](Images/Pasted%20image%2020231108141837.png)

Using `--packet-trace -Pn -n --disable-arp-ping` disables ICMP echo, DNS, and ARP pings while also showing all the packets sent and received to verify the response from the host. In this example I can see that the host responded (in the RCVD line) with a TCP packet containing the flags RA, meaning it responded with the RST and ACK packets signaling that the port was indeed closed. 

#### TCP Connect Scan

To get more accurate results on if a port is open or not, I use `-sT` to enable a TCP connection scan on HTTP/HTTPS ports: 

![](Images/Pasted%20image%2020231108142055.png)

#### Filtered Ports

Some ports are displayed as filtered and doing a packet trace on these ports can help determine if the host dropped or rejected the packets I have sent it. 

Doing a packet trace on the filtered port 445 (with the default max retries of 10) shows that the overall scan time took much longer than usual at 2.11 seconds, meaning that the packet was likely dropped: 

![](Images/Pasted%20image%2020231108142225.png)

If the host's firewall had rejected the packets then the scan time would have been much shorter, for example 0.05 seconds. It would also be likely that I would receive an ICMP reply with type 3 and error code 3 showing that the host would be unreachable. 

#### UDP Scans

Checking for UDP ports is trickier because UDP packets do not require responses like TCP connections do. Any open UDP ports that send responses back are because the application is configured to do so. 

Here I run a scan for open UDP ports and do a packet trace on one that is listed as open: 

![](Images/Pasted%20image%2020231108142505.png)

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

Nmap also gives access to various Lua scripts that can can search for specific info or vulnerabilities. 

To begin using these scripts I choose a host and run the default scripts that Nmap uses with the `-sC` option:

![](Images/Pasted%20image%2020231108150304.png)

I can also choose to run specific script categories like the `vuln` category:

![](Images/Pasted%20image%2020231108150725.png)

Or I could choose to run specific scripts like the `banner` script for banner grabbing: 

![](Images/Pasted%20image%2020231108150902.png)

If it is possible to be more aggressive with the scans, then I can opt to use `-A` to do service detection, OS detection, a traceroute, and run the default scripts:

![](Images/Pasted%20image%2020231108151853.png)

#### Web Server Vulnerability Assessment 

Web servers are some of the most common targets because they are made public to users. Using NSE scripts it is quick and easy to see if a server has known vulnerabilities that can be exploited: 

![](Images/Pasted%20image%2020231108155257.png)

From this scan using the `vuln` script category, I can see that the web server has a public file **robots.txt**. I can view this page in my browser to gain potentially valuable information:

![](Images/Pasted%20image%2020231108155035.png)

### Performance

Scanning and enumerating large networks can take a long time and may be difficult with limited bandwidth. Nmap provides options to control settings like timeout times, maximum number of retries, and maximum number of simultaneous packets being sent. 

To see how performance can be optimized with these options I start by running a default fast scan on a range of IPs: 

![](Images/Pasted%20image%2020231108182638.png)
![](Images/Pasted%20image%2020231108182711.png)

Then, I modify the scan by changing the initial RTT (round time trip) timeout to be 50ms and the maximum to be 100ms. 

![](Images/Pasted%20image%2020231108182857.png)
![](Images/Pasted%20image%2020231108182914.png)

The results show that the overall scan time was significantly faster at 2.62 seconds and in this example found the same amount of hosts as the default fast scan. In other examples some hosts may not be scanned with the modified timeout settings. 

In whitebox penetration tests, for example, there may be more bandwidth to perform scans. One way to take advantage of this is by increasing the number of simultaneous packets Nmap sends out when scanning.

Changing the number of packets using `--min-rate 300` will also heavily reduce the overall scan time compared to the default fast scan:

![](Images/Pasted%20image%2020231108185715.png)
![](Images/Pasted%20image%2020231108185732.png)

![](Images/Pasted%20image%2020231108185750.png)
![](Images/Pasted%20image%2020231108185805.png)

If the case were that the options weren't able to be modified as we would like, there are more general ways to change the timing and aggressiveness of the scan with the `-T` option.

`-T5` is the fastest but also the most aggressive scan type:

![](Images/Pasted%20image%2020231108191831.png)
## Bypass Security Measures

Now that I have completed many different scans on the target hosts and tried various options that Nmap provides to modify its default scans, I will now test different stealth options. If this were a live scenario it would be necessary to attempt to avoid security measures put in place by the target such as firewall and IDS/IPS rules. 

In this section I will compare the results of the default Nmap scans versus some of its more stealthy options. 

### Firewall and IDS/IPS Evasion

Some of the stealth options that Nmap provides are related to the types of packets that Nmap sends when scanning for ports. Sending TCP ACK packets is considered to be much more stealthy as it can get past security rules because the target system can't determine if the connection was initiated internally or externally. 

#### Stealth Scans

First, I compare the results of an ACK scan and the default SYN scan, `-sA` versus `-sS`

![](Images/Pasted%20image%2020231109165452.png)
![](Images/Pasted%20image%2020231109165848.png)

These results show that the RCVD packets change because instead of the target host responding with SYN/ACK or RST and ACK packets it will only respond with RST packets in response to the ACK packets that Nmap sent. 

#### Decoys

If it is known that the target host will block certain subnets or has an IPS in place that will block any connections, then Nmap provides the option to send decoy packets with randomly generated IP addresses. 

I use the option `-D RND:5` to send SYN packets to the target on port 80 but with 5 randomly generated IP addresses:

![](Images/Pasted%20image%2020231109172815.png)

Nmap will automatically mix in the actual IP that I use to perform the scan. 

#### Changing IP Addresses

Now I will try to detect a hosts OS normally and then try with a specified IP address:

![](Images/Pasted%20image%2020231109174137.png)
![](Images/Pasted%20image%2020231109175039.png)

#### DNS Proxying

Nmap typically performs reverse DNS resolution to learn more about the host. Nmap allows the option to specify DNS servers and source ports to attempt to get past misconfiguration of IDS/IPS filters. 

An example of this would be using `--source-port` to send the packet from the DNS port 53 even though I am trying to scan port 445. 

![](Images/Pasted%20image%2020231109180153.png)

## Security Evasion

Now that I have experimented with many of the tools that Nmap provides to learn more about hosts and how to modify its scans to be more stealthy, I will test these methods on three example targets. 

### Target 1 - Easy

For this target I am tasked with finding the OS that the host system is using. There is an IDS/IPS in place that will block my IP address for 3 minutes if a certain number of alerts has been reached. I have been provided with the status page that shows me the number of alerts that have been raised: 

![](Images/Pasted%20image%2020231109185750.png)

This IDS/IPS will record alerts very quickly if I try aggressive scans like doing a simple `sudo nmap 10.129.89.236 -A`. 

Some of the ports that I want to investigate for OS information are 22, 80, 139, and 445. Ports 22 and 80 could easily reveal the OS being used and ports 139 and 445 could potentially verify that it is a Windows host. 

I first started with a stealthy TCP ACK scan to see some of the top ports:

![](Images/Pasted%20image%2020231109190326.png)

Seeing as ports 22 and 80 were both unfiltered I thought that these may be my best options, so I did service scans on both:

![](Images/Pasted%20image%2020231109190456.png)
![](Images/Pasted%20image%2020231109190435.png)

Both came back with Linux Ubuntu as their OS and after confirming that ports 139 and 445 were unresponsive I submitted and confirmed that the OS was Ubuntu. 

### Target 2 - Medium

For this target I want to find out the server version for the target's DNS server. To do some initial reconnaissance on the port I run a TCP ACK scan on port 53 with a packet trace: 

![](Images/Pasted%20image%2020231110131827.png)

The port is listed as filtered and I did not receive any RST packets in response to the ACK packets sent. 

Looking through the Nmap documentation I found there is a script specifically for DNS server version discovery, `dns-nsid`:

![](Images/Pasted%20image%2020231110132140.png)

The results show the intended banner in the **bind.version** section. Running this was very fast and did not raise any significant amount of alerts from the security controls:

![](Images/Pasted%20image%2020231110132220.png)

### Target 3 - Hard

This target had the additional info that the admins were required to enable a service that was valuable to their clients because they required a "large amount of data", and my goal was to find the version that this service was using. 

There were a few ports that I thought could be relevant:
- 22 - file transfers of the data
- 53 - DNS 
- 80/443 - web files
- 445 - Windows file sharing
- 3306 - MySQL server
- 50000 - IBM cloud database
- 27017 - MongoDB

First I did a basic scan to see which ports were open and found that only SSH and HTTP were:

![](Images/Pasted%20image%2020231110162749.png)

I was able to find the versions of these services, but these were not the expected answer. Instead, I focused on the many filtered ports considering that in this scenario the target was known to have setup more robust IDS/IPS filters. 

The only ports that I had tried that were filtered instead of closed were ports 53, 443, 445, and 50000. For these ports I first attempted to modify the scan to use my **eth0** interface and IP address with the options `-S 194.113.74.78 -e eth0`. Unfortunately this did not result in any different results for these ports as no version could be detected because of the filters: 

![](Images/Pasted%20image%2020231110164409.png)

My next goal was to attempt to perform a DNS proxy with `--source-port 53` on these ports to see if using port 53 would be an accepted connection rather than being filtered. Doing this on port 50000 resulted in the port returning as open: 

![](Images/Pasted%20image%2020231110164456.png)

In this scan I used a TCP SYN packet scan and with the modified source port of 53 the target responded with the SYN/ACK packet to signal that the DNS proxy worked. 

Now that I could successfully form a connection with the host I attempted to find the service version of port 50000 through a couple of Nmap scripts specific to IBM DB2, `--script db2-das-info` and `--script broadcast-db2-discover`, but these did not return the expected version flag: 

![](Images/Pasted%20image%2020231110164931.png)

Finally, I tried to check for banners using a netcat connection to port 50000 with the source port of 53:

![](Images/Pasted%20image%2020231110165123.png)

Forming the connection was successful and after a few seconds the expected version flag was output!
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

