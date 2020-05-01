---
title: "Port Scanning"
weight: 103
---
# Port Scanning

Let's continue narrowing down our attack vector. It is time now to grab your host or list of hosts and scan for open ports and running services.  
Port scanners can be categorized between **synchronous or connection-oriented** and **asynchronous or connectionless**.  
Synchronous scanners, like [Nmap](https://nmap.org/), wait for a host response (until the timeout period expires) to determine if the port it's alive. This translates to slower scans, especially on big network ranges, but an accurate port discovery in comparison with connectionless scanners.  
Asynchronous scanners like [scanrad](http://www.vulnerabilityassessment.co.uk/scanrand.htm), [zmap](https://github.com/zmap/zmap), or [masscan](https://github.com/robertdavidgraham/masscan), on the other hand, don't wait for a host response as they create separate threads for each port. This allows a high-speed scan, but less accurate since they cannot detect dropped packets.  
Combining both tools could help you find a balance between speed and accuracy for each engagement. [^captmeelo-benchmark]

## Network Sweep

```sh
sudo masscan -p 7,9,13,21-23,25-26,37,53,79-81,88,106,110-111,113,119,135,139,143-144,179,199,389,427,443-445,465,513-515,543-544,548,554,587,631,646,873,990,993,995,1025-1029,1110,1433,1720,1723,1755,1900,2000-2001,2049,2121,2717,3000,3128,3306,3389,3986,4899,5000,5009,5051,5060,5101,5190,5357,5432,5631,5666,5800,5900,6000-6001,6646,7070,8000,8008-8009,8080-8081,8443,8888,9100,9999-10000,32768,49152-49157 --rate 50000 --wait 0 --open-only -oG masscan.gnmap 10.0.0.0/24 
```

{{<details "Used masscan options">}}
- `-p <ports>`: port(s) to be scanned.
- `--rate <rate>`: rate for transmitting packets (packet per seconds).
- `--wait <seconds>`: seconds after transmit is done to wait for receiving packets before exiting. Default 10 seconds.
- `--open-only`: report only open ports.
- `-oG <filename>`: output scan in grepable format to the given filename.
{{</details>}}

## Host Scan

### TCP

#### Fast 1000 ports scan
```sh
sudo nmap -v -sV -sC -O -T4 -n -Pn -oA nmap_scan 10.0.0.10
```

#### Fast all ports scan
```sh
sudo nmap -v -sV -sC -O -T4 -n -Pn -p- -oA nmap_fullscan 10.0.0.10
```

### UDP

During a UDP scan (`-Su`) `-sV` will send protocol-specific probes, aka _nmap-service-probes_, to every `open|filtered` port. In case of response the state change to `open`. [^nmap-udp]

```sh
sudo nmap -sU -sV --version-intensity 0 -n 10.0.0.0/24
```

{{<details "Used Nmap options">}}
- `-v`: verbose mode.
- `-sU`: UDP scan.
- `-sV`: determine service/version info.
- `-sC`: deter
- `-O`: enable OS detection.
- `-p <port ranges>`: scan specific ports.
- `--open`: only show open ports.
- `-Pn`: no ping.
- `-n`: no DNS resolution.
- `-T<0-5>`: timing template.
- `oX <filename>`: output scan in XML format to the given filename.
- `--version-intensity 0`: try only the probes most likely to be effective.
{{</details>}}

## Reference

[^captmeelo-benchmark]: Capt. Meelo. “Finding the Balance Between Speed & Accuracy During an Internet-Wide Port Scanning - Hack.Learn.Share.” Hack.Learn.Share, 29 July 2019, https://captmeelo.com/pentest/2019/07/29/port-scanning.html.
[^highon-portscanning]: “Penetration Testing Tools Cheat Sheet.” HighOn.Coffee • Security Research • Penetration Testing Blog, https://highon.coffee/blog/penetration-testing-tools-cheat-sheet/#nmap-commands.
[^nmap-udp]: “UDP Scan (-SU) | Nmap Network Scanning.” Nmap: The Network Mapper - Free Security Scanner, https://nmap.org/book/scan-methods-udp-scan.html.
