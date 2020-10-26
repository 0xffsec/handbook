---
menu: "DNS - 53"
title: "DNS (Domain Name Systems) Service Enumeration"
weight: 53
---
# DNS (Domain Name Systems)

## At a Glance

{{<highlight>}}
**Default Port:** 53
{{</highlight>}}

The Domain Name System (DNS) is a hierarchical and decentralized naming system for computers, services, and other resources connected to a network.
It associates different information with domain names assigned to each of the participating entities. Most prominently, it translates more easily memorized domain names to the numerical IP addresses needed for identifying computer services and devices with the underlying network protocols.  [^wiki-dns]

## TCP and UDP

By default, DNS uses UDP on port 53 to serve requests. When the size of the request, or the response, exceeds the single packet size of 512 bytes, the query is re-sent using TCP. Multiple records responses, IPv6 responses, big TXT records, DNSSEC responses, and **zone transfers** are some examples of these requests.

{{<note>}}
When DNS is running on TCP, it is worth checking if [zone trasfer]({{< ref "#zone-transfer" >}}) is enabled.
{{</note>}}

## Banner Grabbing

DNS does not provide an information banner _per se_ but BIND DNS exposes its version by default. [^dns-banner-grabbing]

{{<note>}}
The `version.bind` directive is stored under the `options` section in the `/etc/named.conf` configuration file.
{{</note>}}

#### dig

```sh
dig version.bind CHAOS TXT @{{< param "war.rhost" >}}
```

#### [dns-nsid](https://nmap.org/nsedoc/scripts/dns-nsid.html) NSE Script

```sh
nmap -sV --script dns-nsid -p53 -Pn {{< param "war.rhost" >}}
```

## DNS Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

## Zone Transfer

DNS reconnaissance is especially valuable during the information gathering stage since it could reveal crucial information about the domain and the infrastructure. But it can also reveal new attack vectors, for example, if Virtual Routing is enabled.

A zone transfer is the process where a DNS server, usually a Master server, passes a copy of a **zone** to another DNS server, usually a Slave server. Ideally, these transfers are limited to certain IPs, but misconfigured servers allow these transfers to anyone _asking_ for them.

### dig
```sh
dig axfr @{{< param "war.rhost" >}} domain
```
{{<details "Parameters">}}
- `axfr`: initiate an *AXFR* zone transfer query.
- `@{{< param "war.rhost" >}}`: name or IP of the server to query.
- `domain`: name of the resource record that is to be looked up.
{{</details>}}

{{<note>}}
It is worth trying to initiate a zone transfer without a domain.
{{</note>}}

## Configuration files

Examine configuration files.[^0daysec-enum]

```
host.conf
resolv.conf
named.conf
```

## Further Reading

- [SANS Securing DNS Zone Transfer](https://www.sans.org/reading-room/whitepapers/dns/securing-dns-zone-transfer-868)
- [What the internet looke like in 1982](https://blog.ted.com/what-the-internet-looked-like-in-1982-a-closer-look-at-danny-hillis-vintage-directory-of-users/)

[^wiki-dns]: Contributors to Wikimedia projects. “Domain Name System - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 20 Aug. 2001, https://en.wikipedia.org/wiki/Domain_Name_System.
[^0daysec-enum]: “Penetration Testing Methodology” 0DAYsecurity.Com, http://www.0daysecurity.com/penetration-testing/enumeration.html.
[^dns-banner-grabbing]: “Determining BIND DNS Version Using Dig.” OSI Security - Penetration Testing & Web Application Security Consultants, https://www.osi.security/blog/determining-bind-dns-version-using-dig.
