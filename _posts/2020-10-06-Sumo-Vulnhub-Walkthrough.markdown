---
layout: post
title:  "Sumo: 1 Vulnhub Walkthrough"
date:   2020-10-06 11:56:07 +0530
categories: Walkthroughs Vulnhub
---



## INFO
- User:
  - Difficulty: Intermediate
  - CTFey: No
- Root:
  - Difficulty: Easy
  - CTFey: No
- OS: Linux

## Discovering The Target Machine
We can discover the target by using Netdiscover but generally I use Namp ping-only scan because it takes much less time than Netdiscover. Netdiscover uses ARP packets to 
discover the hosts over a range which can be extremely useful when the target host is blocking ping probes and Nmap ping-only scan won't be able to discover the host. 
<br />That's why I always start with Nmap and in case I don't find anything I move to Netdiscover.
<br />We can use either of the following commands.
```
$nmap -sn -T4 192.168.67.*
OR
$netdiscover -i eth0 -r 192.168.67.1/24 
```
<br />## Screenshot[dicovering host]<br />
We've discovered that the target machine is at [IP]. Let's go and scan that!.<br />
****
## Scanning The Target
```
$nmap -Pn -T4 -sC -sV -n [IP]
```
The Nmap scan tells us that there are 2 ports open **22** and **80**. The version of the services doesn't look vulnerable, Let's explore what's there on port 80.
<br /> -> Screenshot nmap scan <-
<br />
<br />
## Enumeration
->[screenshot website default page]
<br />The website looks like running the default index.html file of web-server. Still there can be useful directory or file or there can be virtual hosting enabled. 
But Virtual Hosting requires the domain name of the target's site to reach the website.<br \>We didn't see any such words till now that can be tried as possible domain names so 
we will go with fuzzing the root directory.
<br />[Screenshot of fuzzing]
<br />**After doing recursive fuzzing with dirbuster we'll get only one interesting directory cgi-bin containing test.sh and /test/test.cgi.**
<br />
<br />
## Exploitation
Because there no such assets on the target other than cgi scripts, we'll go and check whether the target system is vulnerable to **shellshock** vulnerability.
<br />Shellshock is a bash vulnerability and it's advantage can be taken only when some user input or HTTP headers are being processed by bash, perl, python or 
cgi scripts. You can find about shellshock in more detail at [link].
<br />
So we will fire up Burp-Suite and try to exploit by sending a reverse shell, by **changing the user-agent or any other value in HTTP request header**.
<br /> [screenshot burp-suite with shellshock header]
<br />I've started netcat server to accept any connections for reverse shell.
<br />[netcat screenshot with conntect]
We got our reverse shell, that means the system is vulnerable to shell shock. We can also check it pinging to our system and capture icmp packets with tcmdump or 
wireshark.
<br />
Let us now get a proper pty shell with **python3 pty module** and exporting the **TERM env variable**.
<br />[screenshot pty shell]
<br />
<br />
## Privilege Escalation
So now we will head to escalate out privileges and get the system.<br />
We'll check if the system is vulnerable to any kernel exploit with **linux-exploit-suggester**.
<br />[screenshot linux-exploit-suggester]
<br /> 
Seems like it is vulnerable to dirty cow. I'll transfer the exploit to the host machine and compile it there and execute with the following commands:
```
$gcc -pthread c0w.c -o c0w
$./c0w 1
```
[screenshot of final state]<br />
Great! We elevated our privileges. Just need to confirm that will `id`.
<br />
<br />

### ***Please feel free to contact me for any questions, suggestions or constructive critisism.***
