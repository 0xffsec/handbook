---
menu: "Subdomain Enumeration"
title: "Subdomain Enumeration: The Ultimate Guide"
description: "The ultimate subdomain enumeration guide for bugbounty and security assestments."
---

# Subdomain Enumeration

## At a Glance

Sub-domain enumeration is
the process of finding sub-domains
for one or more domains.
It helps to broader the attack surface,
find hidden applications,
and forgotten subdomains.

{{<note>}}
Vulnerabilities tend to be present
across multiple domains and applications
of the same organization.
{{</note>}}

#### Passive Enumeration

- [Certificate Transparency](#certificate-transparency)
- [Google Dorking](#google-dorking)
- [DNS Aggregators](#dns-aggregators)
- [ASN Enumeration](#asn-enumeration)
- [Subject Alternate Name (SAN)](#subject-alternate-name-san)
- [Rapid7 Forward DNS dataset](#rapid7-forward-dns-dataset)

#### Active Enumeration

- [Brute Force Enumeration](#brute-force-enumeration)
- [Zone Transfer](#zone-transfer)
- [DNS Records](#dns-records)
- [Content Security Policy (CSP) Header](#content-security-policy-csp-header)

## Tools

#### Amass [^amass]
```sh
amass enum -passive -d {{< param "war.rdomain" >}} -o results.txt
```
{{<details "Parameters">}}
- `--passive`: Disable DNS resolution names and dependent features.
- `-d`: Domain names separated by commas.
- `-o`: Path to the text file containing terminal `stdout`/`stderr`.
{{</details>}}

#### sublist3r [^sublist3r]
```sh
sublist3r -d {{< param "war.rdomain" >}}
```
## Certificate Transparency

Certificate Transparency (CT) is an Internet security standard
and open-source framework
for monitoring and auditing digital certificates.
It creates a system of public logs
to record all certificates issued by publicly trusted CAs,
allowing efficient identification of mistakenly
or maliciously issued certificates.[^certificate-transparency]

Some CT Logs search engines:

- https://crt.sh/
- https://censys.io/
- https://developers.facebook.com/tools/ct/
- https://google.com/transparencyreport/https/ct/

#### massdns [^massdns]
```sh
./scripts/ct.py {{< param "war.rdomain" >}} | ./bin/massdns -r ./lists/resolvers.txt -o S -w results.txt
```
{{<details "Parameters">}}
- `-r`: Text file containing DNS resolvers.
- `-o`: Flags for output formatting.
- `-w`: Write to the specified output file.
{{</details>}}

#### `crt.sh` PostgreSQL Interface

```sh
#!/bin/sh

query="SELECT ci.NAME_VALUE NAME_VALUE FROM certificate_identity ci WHERE ci.NAME_TYPE = 'dNSName' AND reverse(lower(ci.NAME_VALUE)) LIKE reverse(lower('%.$1'));"

(echo $1; echo $query | \
    psql -t -h crt.sh -p 5432 -U guest certwatch | \
    sed -e 's:^ *::g' -e 's:^*\.::g' -e '/^$/d' | \
    sed -e 's:*.::g';) | sort -u
```

## Google Dorking

Use the `site:` operator to filter
results for the given domain.
Generally,
"`*`" matches any token,
meaning,
an entire term without spaces.

```text
site:*.{{< param "war.rdomain" >}}
```

{{<note>}}
Other search engines,
like [Bing](https://www.bing.com) and [DuckDuckGo](http://www.duckduckgo.com),
offer similar advanced search operators:
- [Bing Advanced Search Keywords](https://help.bing.microsoft.com/#apex/bing/en-us/10001/-1)
- [DuckDuckGo Search Syntax](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/)
- [Google Advanced Search Operators](https://docs.google.com/document/d/1ydVaJJeL1EYbWtlfj9TPfBTE5IBADkQfZrQaBZxqXGs/edit)
{{</note>}}

## DNS Aggregators

- [VirusTotal Passive DNS replication](https://www.virustotal.com/)[^vt-passive-dns]
- [DNSDumpster](https://dnsdumpster.com/)
- [Netcraft - Search DNS](https://searchdns.netcraft.com/)

## ASN Enumeration

Refer to [ASN Enumeration]({{< ref "assets-discovery#asn-enumeration" >}})

## Subject Alternate Name (SAN)

Subject Alternative Name (SAN) is an extension to X.509
that allows additional values
to be associated
with an SSL certificate.
These values, or Names, include
email addresses, URIs, DNS names,
directory names,
and more.
[^openssl-san]

#### OpenSSL [^openssl]
```sh
true | openssl s_client -connect {{< param "war.rdomain" >}}:443 2>/dev/null | openssl x509 -noout -text  | perl -l -0777 -ne '@names=/\bDNS:([^\s,]+)/g; print join("\n", sort @names);'
```
{{<details "Parameters">}}
- `s_client`:  SSL/TLS client program.
- `x509`: output a [x590 structure](https://en.wikipedia.org/wiki/X.509) instead of a certificate request.
- `-noout`: Inhibits the output of the encoded version of the parameters.
- `-text`: Prints out the EC parameters in human readable form.
{{</details>}}


## Rapid7 Forward DNS dataset

The dataset contains
the responses to DNS requests
for all forward DNS names
known by [Rapid7's Project Sonar](https://www.rapid7.com/research/project-sonar/).

[Download Rapid7 Forward DNS datasets](https://opendata.rapid7.com/sonar.fdns_v2/).


## Brute Force Enumeration

Useful Wordlists:

- Jhaddix's [all.txt](https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056)
- Daniel Miessler's [DNS Discovery](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS).
- [Commonspeak2](https://github.com/assetnote/commonspeak2-wordlists)

#### Amass [^amass]
```sh
amass enum -brute -w subdomains.txt -d {{< param "war.rdomain" >}} -o results.txt
```
{{<details "Parameters">}}
- `-brute`: Execute brute forcing after searches.
- `-w`: Path to wordlist file.
- `-d`: Domain names separated by commas.
- `-o`: Path to the text file containing terminal `stdout`/`stderr`.
{{</details>}}

#### massdns [^massdns]
```sh
./scripts/subbrute.py subdomains.txt {{< param "war.rdomain" >}} | ./bin/massdns -r ./lists/resolvers.txt -o S -w results.txt
```
{{<details "Parameters">}}
- `-r`: Text file containing DNS resolvers.
- `-o`: Flags for output formatting.
- `-w`: Write to the specified output file.
{{</details>}}

#### gobuster [^gobuster]
```sh
gobuster dns -t 30 -w subdomains.txt -d {{< param "war.rdomain" >}}
```
{{<details "Parameters">}}
- `dns`: DNS subdomain bruteforcing mode.
- `-d`: target domain.
- `-t <n>`: number of concurrent threads (default 10).
- `-w <wordlist>`: path to the wordlist.
{{</details>}}

## DNS

### Zone Transfer

DNS zone transfer is one mechanism
to replicate DNS databases across DNS servers.

{{<note>}}
Data transfer process
begins by the client
sending a AXFR query
to the server.
{{</note>}}

More under DNS Service Enumeration [Zone Transfers]({{< ref "dns#zone-transfer" >}})

#### dnsrecon [^dnsrecon]
```sh
dnsrecon -a -d tesla.com
```
{{<details "Parameters">}}
- `-a`: Perform AXFR with standard enumeration.
- `-d`: Domain.
{{</details>}}

#### dig

```sh
dig axfr @ns1.{{< param "war.rdomain" >}} {{< param "war.rdomain" >}}
```
{{<details "Parameters">}}
- `axfr`: initiate an *AXFR* zone transfer query.
- `@ns1.{{< param "war.rdomain" >}}`: name or IP of the server to query.
- `{{< param "war.rdomain" >}}`: name of the resource record that is to be looked up.
{{</details>}}


### CNAME Records

A Canonical NAME record (CNAME)
is a type of DNS record
that maps one domain name (an alias)
to another (the canonical name).

CNAMEs may reveal
an organization's sub-domains
and information about running services.

#### dig
```sh
dig +short -x `dig +short {{< param "war.rdomain" >}}`
```
{{<details "Parameters">}}
- `+short`: Provide a terse answer.
- `-x <addr>`: Simplified reverse lookups, for mapping addresses to names.
{{</details>}}

### SPF Records

A Sender Policy Framework record (SPF)
is a type of TXT DNS record
used as an email authentication method.
It specifies the mail servers
authorized to send emails for your domain.

SPF helps protect domains from spoofing.

{{<note>}}
Applications may have internal netblocks
listed in their SPF record.
{{</note>}}

#### dig
```sh
dig +short TXT {{< param "war.rdomain" >}}
```
{{<details "Parameters">}}
- `+short`: Provide a terse answer.
{{</details>}}

## HTTP Headers

### Content Security Policy (CSP)

The HTTP `Content-Security-Policy` response header
allows website administrators
to control resources the browser is allowed to load
for a given page.

{{<note>}}
There are deprecated forms of CSP headers,
they are `X-Content-Security-Policy` and `X-Webkit-CSP`
{{</note>}}

#### curl
```sh
curl -I -s -L https://www.maxrodrigo.com | grep -iE 'Content-Security|CSP'
```
{{<details "Parameters">}}
- `-I`: Fetch the headers only.
- `-s`: Quiet mode.
- `-L`: Follow 3XX redirections.
{{</details>}}

## Further Reading

- [What is Certificate Transparency](https://www.certificate-transparency.org/what-is-ct)
- [The Art of Subdomain Enumeration](https://appsecco.com/books/subdomain-enumeration/)
- [How AXFR Protocol Works](https://cr.yp.to/djbdns/axfr-notes.html)
- [OSINT Through Sender Policy Framework (SPF) Records](https://blog.rapid7.com/2015/02/23/osint-through-sender-policy-framework-spf-records/)

[^certificate-transparency]: “What Is Certificate Transparency?” Certificate Transparency, https://www.certificate-transparency.org/what-is-ct.
[^vt-passive-dns]: “VirusTotal += Passive DNS Replication .” VirusTotal Blog, https://blog.virustotal.com/2013/04/virustotal-passive-dns-replication.html.
[^openssl-san]: OpenSSL Foundation, Inc. “X509v3_config.” OpenSSL, https://www.openssl.org/docs/manmaster/man5/x509v3_config.html#Subject-Alternative-Name.
[^openssl]: OpenSSL Foundation, Inc. “/Docs/Manmaster/Man1/Openssl.Html.” OpenSSL.Org, https://www.openssl.org/docs/manmaster/man1/openssl.html.
[^forward-dns]: Rapid7. “Forward DNS · Rapid7/Sonar Wiki.” GitHub, https://github.com/rapid7/sonar/wiki/Forward-DNS.
[^gobuster]: Reeves, OJ. “GitHub - OJ/Gobuster.” GitHub, https://github.com/OJ/gobuster.
[^massdns]: blechschmidt. “GitHub - Blechschmidt/Massdns: A High-Performance DNS Stub Resolver for Bulk Lookups and Reconnaissance.” GitHub, https://github.com/blechschmidt/massdns.
[^dnsrecon]: darkoperator. “GitHub - Darkoperator/Dnsrecon: DNS Enumeration Script.” GitHub, https://github.com/darkoperator/dnsrecon.
[^sublist3r]: aboul3la. “GitHub - Aboul3la/Sublist3r: Fast Subdomains Enumeration Tool for Penetration Testers.” GitHub, https://github.com/aboul3la/Sublist3r.
[^amass]: “GitHub - OWASP/Amass: In-Depth Attack Surface Mapping and Asset Discovery.” GitHub, https://github.com/OWASP/Amass.
