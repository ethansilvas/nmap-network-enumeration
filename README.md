# Nmap Network Enumeration

In this project I use [Nmap](https://nmap.org/), a vital part of a cybersecurity analysts tool kit, and experiment with the many tools that it provides for mapping out a target network. 

I first go through the core aspects of Nmap in [Host Enumeration](#host-enumeration) such as host discovery, port scanning, service detection, OS detection, and Nmap Scripting Engine. With each of these tools I will test and compare the different options that each command provides to learn about the varying ways to approach a target and optimize your scans. 

Next, in [Bypass Security Measures](#bypass-security-measures), I go over some of the options to detect and evade firewalls and IDS/IPS security rules. Many real-life penetration testing scenarios will have strong security in place to detect network scanning, and I will focus on using stealth scan techniques to get around these controls. 

Finally, in [Security Evasion CTFs](#security-evasion-ctfs), I use the techniques that I demonstrated in all of the previous sections to complete three capture the flag exercises on VM hosts that have IDS/IPS systems in place.  

## Table of Contents

- [Host Enumeration](#host-enumeration)
  - [Host Discovery](#host-discovery)
    - [Scanning a Network Range](#scanning-a-network-range)
    - [Scan Single IP Addresses](#scan-single-ip-addresses)
  - [Host and Port Scanning](#host-and-port-scanning)
    - [Scanning Top Ports](#scanning-top-ports)
    - [TCP Connect Scan](#tcp-connect-scan)
    - [Filtered Ports](#filtered-ports)
    - [UDP Scans](#udp-scans)
    - [Scanning for Versions](#scanning-for-versions)
  - [Saving the Results](#saving-the-results)
  - [Service Enumeration](#service-enumeration)
    - [Banner Grabbing](#banner-grabbing)
  - [Nmap Scripting Engine](#nmap-scripting-engine)
    - [Web Server Vulnerability Assessment](#web-server-vulnerability-assessment)
  - [Performance](#performance)
- [Bypass Security Measures](#bypass-security-measures)
  - [Firewall and IDS/IPS Evasion](#firewall-and-idsips-evasion)
    - [Stealth Scans](#stealth-scans)
    - [Decoys](#decoys)
    - [Changing IP Addresses](#changing-ip-addresses)
    - [DNS Proxying](#dns-proxying)
- [Security Evasion CTFs](#security-evasion-ctfs)
  - [Target 1 - Easy](#target-1---easy)
  - [Target 2 - Medium](#target-2---medium)
  - [Target 3 - Hard](#target-3---hard)

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

## Security Evasion CTFs

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

