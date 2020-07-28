---
title: "IMAP - 143,993"
description: "IMAP (Internet Message Access Protocol) Enumeration for Pentesting"
weight: 143

service: IMAP
service_description: Internet Message Access Protocol
service_port: 143 / 993
---
IMAP is an application-layer protocol
used by email clients
to retrieve messages from a mail server.
It was designed to
manage multiple email clients,
therefore clients generally
leave messages on the server
until the user explicitly deletes them.

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
