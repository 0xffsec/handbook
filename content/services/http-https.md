---
menu: HTTP/S - 80 / 443
title: "HTTP/S (Hypertext Transfer Protocol Secure) Service Enumeration"
weight: 80
---
# HTTP/S (Hypertext Transfer Protocol / Secure)

## At a Glance

{{<highlight>}}
**Default Ports**
- HTTP: 80
- HTTPS (HTTP over TLS or SSL): 443
{{</highlight>}}

HTTP is an application-level protocol
for distributed hypermedia information systems.
It is the standard protocol that defines
how messages are formatted
and sent across the web.

HTTPS (Hypertext Transfer Protocol Secure) is an extension of HTTP.
In HTTPS, the communication protocol is encrypted using Transport Layer Security (TLS) or,
formerly,
Secure Sockets Layer (SSL).
Therefore,
the protocol is also referred to
as HTTP over TLS or HTTP over SSL.

## Banner Grabbing

HTTP

#### Netcat
```sh
nc {{< param "war.rdomain" >}} 80
```

HTTPS

#### openssl [^openssl]
```sh
openssl s_client -connect {{< param "war.rdomain" >}}:443
```
{{<details "Parameters">}}
- `s_client`:  SSL/TLS client program.
{{</details>}}

## Directory Enumeration

Enumerate files and directories.
Look for **unlinked content**,
**temporary directories**,
and **backups**.
Widely used tools include
[dirbuster](https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project),
[gobuster](https://github.com/OJ/gobuster),
[dirb](https://sourceforge.net/projects/dirb/),
and [Burp Suite](https://portswigger.net/burp).

### Wordlists

Included in [Kali's wordlists package](https://tools.kali.org/password-attacks/wordlists)
under `/usr/share/wordlists`.

- `/dirbuster/directory-list-2.3-medium.txt` ( 1.9M - 220560 lines )
- `/dirbuster/directory-list-2.3-small.txt` ( 709K - 87664 lines )
- `/dirb/common.txt` ( 36K - 4614 lines )
- `/dirb/big.txt` ( 180K - 20469 lines )

Other lists.

- Jhaddix's [`content_discovery_all.txt`](https://gist.github.com/jhaddix/b80ea67d85c13206125806f0828f4d10) ( 5.9M - 373535 lines )
- Daniel Miessler's [Web Content Discovery](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content).

{{<note>}}
For more wordlists
refer to [Wordlists]({{< ref "brute-forcing#wordlists">}})
and [Wordlist Generaion]({{< ref "brute-forcing#wordlist-generation">}}).
{{</note>}}

### gobuster [^gobuster]

```sh
gobuster dir -t 30 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://{{< param "war.rhost" >}}/
```
{{<details "Parameters">}}
- `dir`: directory brute-forcing mode.
- `-t <n>`: number of concurrent threads (default 10).
- `-w <wordlist>`: path to the wordlist.
- `-u <URL>`: target URL.
{{</details>}}

{{<note>}}
- Iterate over the results.
- Include status code 403 (Forbidden Error) and brutefoce these directories.
- Add more file extensions to search for; In `gobuster`: `-x sh,pl`.
{{</note>}}

## Source Code

### Inspect

It is a good habit to take a quick look at the pages' source code, scripts, and console outputs.

To active `View Source`, context-click on the page and select `View Page Source` or with the `Ctrl+U` or `Cmd+U` shortcut.

{{<note>}}
Many browsers include a powerful suite of tools, also known as devtools, to inspect and interact with the target website.
{{</note>}}

### Download

If the target uses an open-source app, downloading its codebase will provide helpful information about configuration files, open resources, **default credentials**, etc.

[^gobuster]: Reeves, OJ. “GitHub - OJ/Gobuster.” GitHub, https://github.com/OJ/gobuster.
[^dirb]: Pinuaga, Ramon. “DIRB .” DIRB Homepage, http://dirb.sourceforge.net/.
[^openssl]: OpenSSL Foundation, Inc. “/Docs/Manmaster/Man1/Openssl.Html.” OpenSSL.Org, https://www.openssl.org/docs/manmaster/man1/openssl.html.
