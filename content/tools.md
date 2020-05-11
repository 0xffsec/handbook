---
title: Tools
bookToc: false
---

# Tools

## dirb

DIRB is a Web Content Scanner. It looks for existing (and/or hidden) Web Objects. It basically works by launching a dictionary based attack against a web server and analyzing the response.

[Official Homepage](http://dirb.sourceforge.net/)

## fping

fping is a program to send `ICMP` echo probes to network hosts, similar to ping, but much better performing when pinging multiple hosts. fping has a very long history: Roland Schemers did publish a first version of it in 1992 and it has established itself since then as a standard tool for network diagnostics and statistics.

[Official Homepage](https://fping.org/)

## FTP

File Transfer Protocol client.

## gobuster

Gobuster is a tool used to brute-force URIs (directories and files), DNS subdomains and Virtual Host names on target web servers.

[GitHub Page](https://github.com/OJ/gobuster)


## masscan

This is an Internet-scale port scanner. It can scan the entire Internet in under 6 minutes, transmitting 10 million packets per second, from a single machine.  
It's input/output is similar to [nmap]({{< ref "tools#nmap" >}}), the most famous port scanner. When in doubt, try one of those features.  
Internally, it uses asynchronous transmissions, similar to port scanners like scanrand, unicornscan, and ZMap. It's more flexible, allowing arbitrary port and address ranges.

[GitHub Page](https://github.com/robertdavidgraham/masscan)

## Medusa

Medusa is intended to be a speedy, massively parallel, modular, login brute-forcer. The goal is to support as many services which allow remote authentication as possible. The author considers following items as some of the key features of this application: 

[Official Homepage](http://foofus.net/goons/jmk/medusa/medusa.html)

## MSFConsole

The most popular interface to the Metasploit Framework (MSF). It provides an “all-in-one” centralized console and allows you efficient access to virtually all of the options available in the MSF.

[Official Homepage](https://www.offensive-security.com/metasploit-unleashed/msfconsole/)

## nbtscan

This is a command-line tool that scans for open `NETBIOS` nameservers on a local or remote TCP/IP network, and this is a first step in finding of open shares. It is based on the functionality of the standard Windows tool `nbtstat`, but it operates on a range of addresses instead of just one. I wrote this tool because the existing tools either didn't do what I wanted or ran only on the Windows platforms: mine runs on just about everything.

[Official Homepage](http://www.unixwiz.net/tools/nbtscan.html)

## Netcat

Netcat is a featured networking utility which reads and writes data across network connections, using the TCP/IP protocol.  
It is designed to be a reliable "back-end" tool that can be used directly or easily driven by other programs and scripts. At the same time, it is a feature-rich network debugging and exploration tool, since it can create almost any kind of connection you would need and has several interesting built-in capabilities.

[Official Homepage](http://netcat.sourceforge.net/)

## netdiscover

Active/passive ARP reconnaissance tool.

[Debian's Reference Manual](https://manpages.debian.org/testing/netdiscover/netdiscover.8.en.html)

## Nmap

Nmap (“Network Mapper”) is an open source tool for network exploration and security auditing. It was designed to rapidly scan large networks, although it works fine against single hosts. Nmap uses raw IP packets in novel ways to determine what hosts are available on the network, what services (application name and version) those hosts are offering, what operating systems (and OS versions) they are running, what type of packet filters/firewalls are in use, and dozens of other characteristics. While Nmap is commonly used for security audits, many systems and network administrators find it useful for routine tasks such as network inventory, managing service upgrade schedules, and monitoring host or service uptime.

[Official Homepage](https://nmap.org/)

## p0f

P0f is a tool that utilizes an array of sophisticated, purely passive traffic fingerprinting mechanisms to identify the players behind any incidental TCP/IP communications (often as little as a single normal `SYN`) without interfering in any way.

[Official Homepage](https://lcamtuf.coredump.cx/p0f3/)

## ping
Send ICMP `ECHO_REQUEST` packets to network hosts.

[FreeBSD System Manager's Manual](https://www.freebsd.org/cgi/man.cgi?query=ping&sektion=8)

## SearchSploit

Command line search tool for [Exploit-DB](https://www.exploit-db.com/) that also allows you to take a copy of Exploit Database with you, everywhere you go. SearchSploit gives you the power to perform detailed off-line searches through your locally checked-out copy of the repository. This capability is particularly useful for security assessments on segregated or air-gapped networks without Internet access.

[Official Homepage](https://www.exploit-db.com/searchsploit)

## Telnet

User interface to TELNET.

[Official Homepage](http://telnet.org/)

## THC Hydra

Hydra is a parallelized login cracker which supports numerous protocols to attack. It is very fast and flexible, and new modules are easy to add. This tool makes it possible for researchers and security consultants to show how easy it would be to gain unauthorized access to a system remotely.

[Github Page](https://github.com/vanhauser-thc/thc-hydra)

