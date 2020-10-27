---
menu: Assets Discovery
title: Assets Discovery
description: The reconnaissance, or information gathering, stage plays a major role during a penetration test or a bug bounty hunting. Assets Discovery helps you identify new targets within the scope. These new seeds, or domains, will broader the attack surface and provide a better understanding of the target and its infrastructure.
---

# Assets Discovery

## At a Glance

Assets Discovery helps you identify new targets within the scope. These new seeds, or domains, will broader the attack surface and provide a better understanding of the target and its infrastructure.

## Acquisitions

Looking for acquisitions may expand the available assets **if they are in scope**.

- [Crunchbase](https://www.crunchbase.com/)
- [Google](https://www.google.com)
- [Wikipedia](https://www.wikipedia.com)

{{<note>}}
Look for newer acquisitions and verify it they are still valid
{{</note>}}

## ASN Enumeration

An Autonomous System Number (ASN) is a unique number
assigned to an Autonomous System (AS).
An Autonomous System is a set of routers,
or IP ranges,
under a single technical administration.

ASNs are assigned in blocks by Internet Assigned Numbers Authority (IANA)
to regional Internet registries (RIRs).
The appropriate RIR then
assigns ASNs to entities within its designated area
from the block assigned by IANA.
[^rfc-1930]

{{<note>}}
These ASN’s will help us picture
the entity’s IT infrastructure.
{{</note>}}

### Online Tools

The most reliable way to get these
is manually through [Hurricane Electric’s](https://www.he.net/) BGP Toolkit:

- http://bgp.he.net

Or thought the regional registries services:

- Africa: [AFRINIC - Regional Internet Registry for Africa](https://www.afrinic.net/)
- Asia: [APNIC - Regional Internet Registry for Asia Pacific](https://www.apnic.net/)
- Europe: [RIPE](https://www.ripe.net/)
- Latin America: [LACNIC - Internet Addresses Registry for Latin America and the Caribbean](https://www.lacnic.net/)
- North America: [ARIN - American Registry for Internet Numbers](https://www.arin.net/about/welcome/region/)

{{<note>}}
Because of the advent of cloud infrastructure,
ASNs may not provide a complete picture of a network.
Assets could also exist on cloud environments like
[AWS](https://aws.amazon.com/), [GCP](https://cloud.google.com/), and [Azure](https://azure.microsoft.com/en-us/).
{{</note>}}

### Automated Tools

#### [OWASP Amass](https://owasp-amass.com/) Intel

Amass `intel` module
collects open-source intelligence for the target organization.
It allows you to find root domain names
associated with it.

```sh
amass intel -org <org-name>
```

```sh
amass intel -asn <asn>
```
{{<details "Parameters">}}
- `intel`: Discover targets for enumeration.
- `-org <name>`: Search `<name>` provided against AS description information.
- `-asn <asn>`: IP and ranges separated by commas.
{{</details>}}

## Reverse WHOIS

A WHOIS domain lookup allows you to
trace the ownership of a domain name
plus additional information such as
expiration date, organization name, emails, addresses, phone numbers.

Reverse WHOIS leverage these registries
and allow us to perform lookups based on
that additional information.[^rfc-1834]

### Online Tools

- [Whoxy Whois API](https://www.whoxy.com/)
- [Reverse Whois Lookup ](https://www.reversewhois.io/)
- [DomainEye Reverse Whois](https://domaineye.com/reverse-whois/)
- [domainIQ - Comprehensive domain name intelligence.](https://www.domainiq.com/)

### Automated Tools

#### [DomLink](https://github.com/vysecurity/DomLink)

Domlink provides a helpful recursive search.

```sh
python domlink.py -A <whoxy-api-key> -C <org-name> -o target.out.txt
```

#### [Whoxy API](https://www.whoxy.com/reverse-whois/)

```sh
curl --no-progress-meter "https://api.whoxy.com/?key=<whoxy-api-key>&reverse=whois&name=<org-name>" | jq
```

## Tracking Codes

Many companies share tracking codes
across different products.
Search for popular services IDs
like Google Analytics or Google Adsense.

### Online Tools

- [BuiltWith Relationship Profile](https://builtwith.com/)
- [Source Code Search Engine](https://publicwww.com/)
- [SpyOnWeb](https://spyonweb.com/)

## Google Dorking

Search for
any company-wide
distinctive text like:

- Copyright text
- Terms of service text
- Privacy Policy text

```txt
"© 2006-2020 Company, Corp." -site:*.domain.com inurl:company
```

{{<note>}}
Other search engines,
like [Bing](https://www.bing.com) and [DuckDuckGo](http://www.duckduckgo.com),
offer similar advanced search operators:
- [Bing Advanced Search Keywords](https://help.bing.microsoft.com/#apex/bing/en-us/10001/-1)
- [DuckDuckGo Search Syntax](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/)
- [Google Advanced Search Operators](https://docs.google.com/document/d/1ydVaJJeL1EYbWtlfj9TPfBTE5IBADkQfZrQaBZxqXGs/edit)
{{</note>}}

## Shodan

Shodan is a search engine
for Internet-connected devices.
Try filter by
organization: `org:<org-name>`
or hostname: `hostname: <domain>`.

{{<note>}}
Shodan usually offers a $5 lifetime membership during Black Friday.
{{</note>}}

[^rfc-1930]: “RFC 1930 - Guidelines for Creation, Selection, and Registration of an Autonomous System (AS).” IETF Tools, https://tools.ietf.org/html/rfc1930#section-3.
[^rfc-1834]: “RFC 1834 - Whois and Network Information Lookup Service, Whois++.” IETF Tools, https://tools.ietf.org/html/rfc1834.
