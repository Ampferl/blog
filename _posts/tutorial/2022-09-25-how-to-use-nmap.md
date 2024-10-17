---
title: Learn Nmap to find your first Network Vulnerability 
date: 2022-09-25 00:00:00 +8000
categories: [Tutorial, Tools]
tags: [tutorial, tools, nmap, enumeration, scanning, security, scripting, ports]
image:
  path: /assets/img/post/nmap-tutorial/Nmap_thumbnail.png 
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Nmap Tutorial
---
 
Learn how to use Nmap, to scan and analyze your network, to find network vulnerabilities. Perform Version and OS detection easily and find open ports.  
[Nmap Cheatsheet](#cheatsheet) - 
[Official Website](https://nmap.org/) -
[GitHub](https://github.com/nmap/nmap)
## What is Nmap?
Network Mapper, alias Nmap, is an open-source tool for network exploration and security auditing.
You can use it to scan large networks pretty fast, but it also works against single hosts. Nmap sends raw IP packets to the target host to gather information and output the list of the scanned hosts. It will determine what services, operating systems, packet filters/firewalls, and dozens of other stuff are running on the scanned system.
## Commands
### Basic Scanning
To create a map of the target network, you have to scan it to list all active devices. To do that, you can use two different techniques:  

- Ping scan - Scans all running devices in a subnet:
```shell
nmap -sP 192.168.1.1/24
```
- Host scan - Scan a single host for the 1024 well-known ports:
```shell
nmap hackerask.com
```
![Nmap Basic Scanning](/assets/img/post/nmap-tutorial/Nmap_basic-scan.png)
To target a single or range of ports you can use the `-p` flag:  

- Scan a single port:
```shell
nmap -p 22 hackerask.com
```
- Scan a range of ports:
```shell
nmap -p 8000-9000 hackerask.com
```
- Scan a list of ports:
```shell
nmap -p 8080,3000,5000 hackerask.com
```
- Scan all ports:
```shell
nmap -p- hackerask.com
```
- Scan the 100 most popular ports using the `-F` flag:
```shell
nmap -F hackerask.com
```
![Nmap 100 Most Popular Ports](/assets/img/post/nmap-tutorial/Nmap_popular-port-scan.png)
To get more information about the scanned host, you can use the verbose output flag `-v`:
```shell
nmap -v hackerask.com
```
That is useful to export nmap results to avoid redundant scans and monitor the step by step actions Nmap performs on a network.
### Stealth Scanning
Stealth scans complicate the detection of the scanning system for a target by not completing the 3-way handshake of a TCP connection.  
It sends an SYN packet to the host. If an SYN/ACK packet is received, a port on the target is open, and you can open a TCP connection.  

To perform a stealth scan, you have to use the flag `-sS`:
```shell
nmap -sS hackerask.com
```
The downside of stealth scanning is that it is slower and not as aggressive as the other types of scanning.
### Version Scanning
Let's find the versions of applications with Nmap, which is a crucial part of penetration testing.  
You can search for existing vulnerabilities in the [Common Vulnerability and Exploits (CVE)](https://cve.mitre.org/) database for the particular found version.

To scan a version of a service you can use the `-sV` flag:  
```shell
nmap -sV hackerask.com
```
But keep in mind, that the version scan is not always correct. But it will give you a good overview of the services on the scanned system.
![Nmap Version Scanning](/assets/img/post/nmap-tutorial/Nmap_version-scan.png)
### Operating System Scanning
Nmap can also provide information about the active operating system on the target host using TCP/IP fingerprints.

You can use the `-O` flag to enable the operating system detection:  
```shell
nmap -O hackerask.com
```
It is also possible to use the flag `-osscan-limit` to limit the detection to promising targets. Nmap will show the probability of each operating system guess.  
![Nmap OS Scanning](/assets/img/post/nmap-tutorial/Nmap_os-scan.png)
Like version scanning, the OS scan is not always accurate but can help to collect some information about the target.
### Aggressive Scanning
But to get far better information than with regular scans, Nmap provides an aggressive mode. That enables OS and version detection, traceroute and script scanning.  
The flag `-A` will enable the aggressive scan:
```shell
nmap -A hackerask.com
```
![Nmap OS Scanning](/assets/img/post/nmap-tutorial/Nmap_aggressive-scan.png)
However, aggressive scans will send more packets to the target host, which is easier detectable.
## Nmap Scripting Engine
The Nmap Scripting Engine(short NSE) is a powerful tool write and automate scans.
### How to perform a scan using the NSE
To perform such a scan use the `--script` flag:
```shell
nmap --script=<script-name> hackerask.com
```
### How to find NSE scripts?
You can find the location of all available NSE scripts using the locate util:
```shell
locate .nse
```
![Nmap OS Scanning](/assets/img/post/nmap-tutorial/Nmap_script-locate.png)
You can also use the list of NSE scripts on the nmap homepage:  
[List of Nmap Scripts](https://nmap.org/nsedoc/scripts/)
### How to use Nmap to scan for Vulnerabilities
To peform a vulnerability scan, Nmap provides the powerful script category <b>vuln</b>. To allow Nmap to use the Vulners exploit database, pass the `-sV` flag while using the NSE script:
```shell
nmap -sV --script=vuln hackerask.com
```
![Nmap Vulnerability Scanning](/assets/img/post/nmap-tutorial/Nmap_vuln-scan.png)


<h2 id="cheatsheet">Cheatsheet</h2>
<div id="cheatsheet">
<h3>Basic Scanning</h3>
    <div class="row">
        <div class="col-md-5">
            <b>Ping Scanning:</b><br>
            <code>nmap -sP &lt;target&gt;</code>
        </div>
        <div class="col-md-5">
            <b>Scan single port:</b><br>
            <code>nmap -p &lt;port&gt; &lt;target&gt;</code>
        </div>
        <div class="col-md-5">
            <b>Scan range of ports:</b><br>
            <code>nmap -p &lt;port&gt;-&lt;port&gt; &lt;target&gt;</code>
        </div>
        <div class="col-md-5">
            <b>Scan all ports:</b><br>
            <code>nmap -p- &lt;target&gt;</code>
        </div>
        <div class="col-md-5">
            <b>Scan 100 most popular ports:</b><br>
            <code>nmap -F &lt;target&gt;</code>
        </div>
    </div>
<h3>Advanced Scanning</h3>
    <div class="row">
        <div class="col-md-5">
            <b>Stealth Scanning:</b><br>
            <code>nmap -sS &lt;target&gt;</code>
        </div>
        <div class="col-md-5">
            <b>Version Scanning:</b><br>
            <code>nmap -sV &lt;target&gt;</code>
        </div>
        <div class="col-md-5">
            <b>OS Scanning:</b><br>
            <code>nmap -O &lt;target&gt;</code>
        </div>
        <div class="col-md-5">
            <b>Aggressive Scanning:</b><br>
            <code>nmap -A &lt;target&gt;</code>
        </div>
    </div>
<h3>Nmap Scripting Engine</h3>
    <div class="row">
        <div class="col-md-5">
            <b>Basic NSE scan:</b><br>
            <code>nmap --script=&lt;script&gt; &lt;target&gt;</code>
        </div>
        <div class="col-md-5">
            <b>Vulnerability Scan:</b><br>
            <code>nmap -sV --script=vuln &lt;target&gt;</code>
        </div>
    </div>
</div>
