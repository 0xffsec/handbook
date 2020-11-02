---
menu: "IMAP - 143 / 993"
title: "IMAP (Internet Message Access Protocol) Service Enumeration"
weight: 143
---
# IMAP (Internet Message Access Protocol)

## At a Glance

{{<highlight>}}
**Default Ports**
- IMAP: 143
- IMAPS (IMAP over SSL): 993
{{</highlight>}}

IMAP is an application-layer protocol
used by email clients
to retrieve messages from a mail server.
It was designed to
manage multiple email clients,
therefore clients generally
leave messages on the server
until the user explicitly deletes them.
[^imap-wiki]

## Banner Grabbing

#### Telnet
```sh
telnet {{< param "war.rhost" >}} 143
```

#### Netcat
```sh
nc -n {{< param "war.rhost" >}} 143
```

#### openssl [^openssl]
```sh
openssl s_client -connect {{< param "war.rhost" >}}:993
```
{{<details "Parameters">}}
- `s_client`:  SSL/TLS client program.
{{</details>}}

## NTLM Information Disclosure

See [SMTP NTLM Information Disclosure]({{< ref "smtp#ntlm-information-disclosure" >}})

#### Manually

```sh
telnet {{< param "war.rdomain" >}} 143
...
>> a1 AUTHENTICATE NTLM
+
>> TlRMTVNTUAABAAAAB4IIAAAAAAAAAAAAAAAAAAAAAAA=
+ TlRMTVNTUAACAAAACgAKADgAAAAFgooCBqqVKFrKPCMAAAAAAAAAAEgASABCAAAABgOAJQAAAA9JAEkAUwAwADEAAgAKAEkASQBTADAAMQABAAoASQBJAFMAMAAxAAQACgBJAEkAUwAwADEAAwAKAEkASQBTADAAMQAHAAgAHwMI0VPy1QEAAAAA
```

#### [imap-ntlm-info](https://nmap.org/nsedoc/scripts/imap-ntlm-info.html) NSE Script

```sh
nmap -p 143,993 --script imap-ntlm-info {{< param "war.rhost" >}}
```

## IMAP Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

[^imap-wiki]: Contributors to Wikimedia projects. “Internet Message Access Protocol - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 7 Sept. 2001, https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol.
[^openssl]: OpenSSL Foundation, Inc. “/Docs/Manmaster/Man1/Openssl.Html.” OpenSSL.Org, https://www.openssl.org/docs/manmaster/man1/openssl.html.
