---
menu: "NNTP - 119 / 433 / 563"
title: "NNTP (Network News Transfer Protocol) Service Enumeration"
weight: 119
---
# NNTP (Network News Transfer Protocol)

## At a Glance

{{<highlight>}}
**Default Ports**
- NNTP: 119
- NNTPS (NNTPS over TLS): 563
- NNSP (server-server bulk transfer): 433
{{</highlight>}}

NNTP is an application-layer protocol
used for transporting [Usenet](https://en.wikipedia.org/wiki/Usenet) news articles
between news servers.
Client applications can also
inquire,
retrieve,
and post articles.

## Banner Grabbing

#### Telnet
```sh
telnet {{< param "war.rhost" >}} 119
```

#### Netcat
```sh
nc -n {{< param "war.rhost" >}} 119
```

#### openssl [^openssl]
```sh
openssl s_client -crlf -connect {{< param "war.rhost" >}}:563
```
{{<details "Parameters">}}
- `s_client`:  SSL/TLS client program.
- `-crlf`:  translate a line feed from the terminal into `CR+LF`.
{{</details>}}

## Commands

Various commands allow clients to retrieve,
send,
and post articles.
The difference between send and post
is that the latter involves
articles with incomplete header information.

NNTP also provides
active and passive news transfer,
or "pushing" and "pulling" respectively.
When pushing,
the client offers an article through `IHAVE <message_id>`.
When pulling,
the client requests a list of available articles
for the specified date
through `NEWNEWS <YYMMDD> <HHMMSS>`.

Several commands also allow clients to retrieve
the article header and body separately,
or even single header lines from a range of articles.
[^oreilly-nntp]

{{<note>}}
NNTP commands responses always end with a period (`.`) on a line by itself.
{{</note>}}

```txt
CAPABILITIES            List server capabilities.
HELP                    Show available commands.
MODE READER             Use Reader mode. Reader mode uses a lot of commands, use HELP.
LIST                    List groups.
SELECT <group>          Select group.
LISTGROUP <group>       List article in a group.
HEAD <article_id>       Retrieve article header.
BODY <article_id>       Retrieve article body.
ARTICLE <article_id>    Retrieve article.
POST                    Post article.
```

## NNTP Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

[^rfc977]: “RFC 977 - Network News Transfer Protocol.” IETF Tools, https://tools.ietf.org/html/rfc977.
[^openssl]: OpenSSL Foundation, Inc. “/Docs/Manmaster/Man1/Openssl.Html.” OpenSSL.Org, https://www.openssl.org/docs/manmaster/man1/openssl.html.
[^oreilly-nntp]: “22. NNTP and the Nntpd Daemon - Linux Network Administrator’s Guide, Second Edition [Book].” O’Reilly Online Learning, O’Reilly Media, Inc., https://www.oreilly.com/library/view/linux-network-administrators/1565924002/ch22.html.
