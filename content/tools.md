---
title: Tools
bookToc: false
---

# Tools

## fping

fping is a program to send ICMP echo probes to network hosts, similar to ping, but much better performing when pinging multiple hosts. fping has a very long history: Roland Schemers did publish a first version of it in 1992 and it has established itself since then as a standard tool for network diagnostics and statistics.

## masscan

This is an Internet-scale port scanner. It can scan the entire Internet in under 6 minutes, transmitting 10 million packets per second, from a single machine.  
It's input/output is similar to nmap, the most famous port scanner. When in doubt, try one of those features.
Internally, it uses asynchronous tranmissions, similar to port scanners like scanrand, unicornscan, and ZMap. It's more flexible, allowing arbitrary port and address ranges.
[GitHub Page](https://github.com/robertdavidgraham/masscan)

## nbtscan

This is a command-line tool that scans for open NETBIOS nameservers on a local or remote TCP/IP network, and this is a first step in finding of open shares. It is based on the functionality of the standard Windows tool nbtstat, but it operates on a range of addresses instead of just one. I wrote this tool because the existing tools either didn't do what I wanted or ran only on the Windows platforms: mine runs on just about everything.

[website](http://www.unixwiz.net/tools/nbtscan.html)

## netdiscover

Active/passive ARP reconnaissance tool.  
[Debian's Reference Manual](https://manpages.debian.org/testing/netdiscover/netdiscover.8.en.html)

## Nmap

Nmap (“Network Mapper”) is an open source tool for network exploration and security auditing. It was designed to rapidly scan large networks, although it works fine against single hosts. Nmap uses raw IP packets in novel ways to determine what hosts are available on the network, what services (application name and version) those hosts are offering, what operating systems (and OS versions) they are running, what type of packet filters/firewalls are in use, and dozens of other characteristics. While Nmap is commonly used for security audits, many systems and network administrators find it useful for routine tasks such as network inventory, managing service upgrade schedules, and monitoring host or service uptime.

[website](https://nmap.org/)

## p0f

P0f is a tool that utilizes an array of sophisticated, purely passive traffic fingerprinting mechanisms to identify the players behind any incidental TCP/IP communications (often as little as a single normal SYN) without interfering in any way.  
[website](https://lcamtuf.coredump.cx/p0f3/)

## ping
Send ICMP `ECHO_REQUEST` packets to network hosts.

[FreeBSD System Manager's Manual](https://www.freebsd.org/cgi/man.cgi?query=ping&sektion=8)
