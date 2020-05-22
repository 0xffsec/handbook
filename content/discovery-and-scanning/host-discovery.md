---
title: "Host Discovery"
weight: 102
---
# Host Discovery

One of the first steps during the network enumeration is to reduce a set of IPs into a list of active or interesting hosts. Depending on if you are inside the network or scanning remotely, how much noise you can make, and your discovery requirements, different tools, and options are available.

## Internal

### Passive

Passive discovery relies on monitoring network layer traffic to detect network topology, services, and applications. The process does not generate any network traffic.  
Although is a slow process compared to an active scan and unreliable for service enumeration, as no packets are injected in the network, there is no risk of unintentional service disruption. Giving the advantage of being able to find intermittently offered or protected services that are often missed by active scanning. [^net-scanning-detection-strategies]

#### netdiscover - ARP [^netdiscover]

```sh
sudo netdiscover -p
```
- `-p`: passive mode. It does not send anything, but **does only sniff**.

#### p0f - Fingerprinting [^p0f]

```sh
sudo p0f -p
```
- `-p`: promiscuous mode; by default, it listens only to packet addressed or routed through.  
        **On IP-enabled interfaces can be detected remotely**.

### Active

In contrast, active discovery does inject a variety of packets into the network, making it well suited for open port discovery and fingerprinting. However, these techniques are not without drawbacks. Scans can be invasive, generate too much noise and, in some cases cause service interruptions due to the type of packets that may be sent.


#### netdiscover - ARP [^netdiscover]

```sh
sudo netdiscover -r {{< param rcidr >}}
```

- `-r <range>`: scan a given range instead of auto scan.

#### Nmap


```sh
nmap -sn {{< param rcidr >}}
```

- `-sn`: No port scan AKA **ping scan**.  [^nmap-host-discovery]
    Consists of an ICMP echo request, TCP SYN to port 443, TCP ACK to port 80, and an ICMP timestamp request by default.

#### nbtscan - NetBIOS [^nbtscan]

```sh
sudo nbtscan {{< param rcidr >}}
```

### ICMP
Refer to [External ICMP Scanning]({{< ref "#icmp-1" >}})

## External

### ICMP

As defined in [RFC 792](https://tools.ietf.org/html/rfc792), ICMP messages are typically used for diagnostic or control purposes or generated in response to errors in IP operations (as specified in [RFC 1122](https://tools.ietf.org/html/rfc1122)).[^data-communications-and-networking]  
Although it is possible to use ICMP requests to discover if a host is up or not, it is common to find all these packets being filtered, making it an unreliable method.

#### ping

```sh
ping -c 1 {{< param rhost >}}
```
- `-c <count>`: stops after `count` replies.

#### fping [^fping]

```sh
fping -g {{< param rcidr >}}
```
- `-g, --generate <target>`: generates target list. `target` can be start and end IP or a CIDR address.

#### Nmap

```sh
nmap -PEPM -sn -n {{< param rcidr >}}
```
{{<details "Parameters">}}
- `-PE; -PP; -PM`: ICMP echo, timestamp, and netmask request discovery probes.
- `-sn`: No port scan.
- `-n`: No DNS resolution.
{{</details>}}

### Port Discovery

Another commonly used technique is port scanning. It allows you to identify running services, consequently, interesting hosts.
See [Port Scanning]({{< ref "port-scanning.md" >}}).

## Reference

[^nmap-host-discovery]: “Host Discovery | Nmap Network Scanning.” Nmap: The Network Mapper - Free Security Scanner, https://nmap.org/book/man-host-discovery.html.
[^data-communications-and-networking]: Forouzan, Behrouz A. Data Communications and Networking. Huga Media, 2007, pp. 621–630.
[^net-scanning-detection-strategies]: Whyte, David. Network Scanning Detection Strategies for Enterprise Networks. Sept. 2008, pp. 12-22 https://pdfs.semanticscholar.org/bb60/dc6cf24ea1f17126511e0998d3c55bdd50f9.pdf.
[^netdiscover]: “GitHub - Netdiscover-Scanner/Netdiscover: Netdiscover, ARP Scanner (Official Repository).” GitHub, https://github.com/netdiscover-scanner/netdiscover.
[^p0f]: Zalewski, Michal. “P0f V3.” [Lcamtuf.Coredump.Cx], https://lcamtuf.coredump.cx/p0f3/.
[^nbtscan]: “NETBIOS Nameserver Scanner.” Steve Friedl’s Home Page, http://www.unixwiz.net/tools/nbtscan.html.
[^fping]: Schweikert, David. “Fping.” Fping Homepage, https://fping.org/.
