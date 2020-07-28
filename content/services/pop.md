---
title: "POP - 110 / 995"
description: "POP (Post Office Protocol) Enumeration for Pentesting"
weight: 110

service: POP
service_description: Post Office Protocol
service_port: 110 / 995
---
POP is an application-layer protocol
used by email clients
to retrieve messages from a mail server.
It provides access via IP to mailboxes
maintained on a server.

Because POP was designed for temporary Internet connection,
clients connect,
retrieve messages,
store them on the client,
and finally delete them from the server.
Clients also have the option
to leave messages on the server.
By contrast,
[IMAP]({{< ref "imap" >}}) was designed to normally leave all messages
on the server
allowing multiple client applications
as online and offline modes.
[^pop3-wiki]

## Banner Grabbing

#### Telnet
```sh
telnet {{< param "war.rhost" >}} 110
```

#### Netcat
```sh
nc -n {{< param "war.rhost" >}} 110
```

#### openssl [^openssl]
```sh
openssl s_client -crlf -connect {{< param "war.rhost" >}}:995
```
{{<details "Parameters">}}
- `s_client`:  SSL/TLS client program.
- `-crlf`:  translate a line feed from the terminal into `CR+LF`.
{{</details>}}

## POP3 Capabilities

POP3 capabilities are defined in [RFC2449](https://tools.ietf.org/html/rfc2449#section-6). The `CAPA` command allows a client to ask a server what commands it supports and possibly any site-specific policy.

#### [pop3-capabilities](https://nmap.org/nsedoc/scripts/pop3-capabilities.html) NSE Script

```sh
nmap -p 110,995 --script pop3-capabilities {{< param "war.rhost" >}}
```

## NTLM Information Disclosure

See [SMTP NTLM Information Disclosure]({{< ref "smtp#ntlm-information-disclosure" >}})

#### [pop3-ntlm-info](https://nmap.org/nsedoc/scripts/pop3-ntlm-info.html) NSE Script

```sh
nmap -p 110,995 --script pop3-ntlm-info {{< param "war.rhost" >}}
```

## POP Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

[^openssl]: OpenSSL Foundation, Inc. “/Docs/Manmaster/Man1/Openssl.Html.” OpenSSL.Org, https://www.openssl.org/docs/manmaster/man1/openssl.html.
[^pop3-wiki]: Contributors to Wikimedia projects. “Post Office Protocol - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 9 Sept. 2001, https://en.wikipedia.org/wiki/Post_Office_Protocol.
