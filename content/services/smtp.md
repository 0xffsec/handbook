---
menu: "SMTP - 25 / 465 / 587"
title: "SMTP (Simple Mail Transfer Protocol) Service Enumeration"
weight: 25
---
# SMTP (Simple Mail Transfer Protocol)

## At a Glance

{{<highlight>}}
**Default Ports**
- SMTP Relay (server-server communication): 25
- SMTP Message Submission (client-server communcation): 578
- SMTPS (Deprecated): 465
{{</highlight>}}

SMTP is a communication protocol
for email transmission.
It is commonly used to
relay and submit messages
to another email servers.

SMTP is a delivery protocol only.
Meaning mail is "pushed"
to a destination mail server,
or next-hop server,
as it arrives.
Mail is routed
based on the destination server,
not individual users
to which it is addressed.
Other protocols,
such as the [Post Office Protocol (POP)]({{< ref "pop">}})
and the [Internet Message Access Protocol (IMAP)]({{< ref "imap" >}})
are specifically designed for use by individual users
retrieving messages and managing mailboxes.
[^rfc-5321]

Communication between sender and receiver
is made by issuing command strings.
See [SMTP commands reference](https://tools.ietf.org/html/rfc5321#section-4.1).

## Banner Grabbing

### SMTP

#### Telnet
```sh
telnet {{< param "war.rhost" >}} 25
```

#### Netcat
```sh
nc -n {{< param "war.rhost" >}} 25
```

### SMPTS

#### openssl [^openssl]
```sh
openssl s_client -starttls smtp -crlf -connect {{< param "war.rhost" >}}:587
```
{{<details "Parameters">}}
- `s_client`:  SSL/TLS client program.
- `-starttls <protocol>`: send the protocol-specific message(s) to switch to TLS for communication.
- `-crlf`:  translate a line feed from the terminal into `CR+LF`.
{{</details>}}

## Enumeration

#### [smtp-commands](https://nmap.org/nsedoc/scripts/smtp-commands.html) NSE Script

```sh
nmap -p 25,465,587 --script smtp-commands {{< param "war.rhost" >}}
```

#### [smtp-enum-users](https://nmap.org/nsedoc/scripts/smtp-enum-users.html) NSE Script

```sh
nmap -p 25,465,587 --script smtp-enum-users {{< param "war.rhost" >}}
```

## NTLM Information Disclosure

On Windows,
with NTLM authentication enabled,
sending a SMTP NTLM authentication request
with null credentials
will cause the remote service
to respond with a NTLMSSP message
disclosing information to include
NetBIOS, DNS,
and OS build version.
[^nse-smtp-nltm-info]
[^ntlm-disclosure]

#### Manually

```sh
telnet {{< param "war.rdomain" >}} 587
...
>> HELO
250 example.com Hello [x.x.x.x]
>>AUTH NTLM 334
NTLM supported
>>TlRMTVNTUAABAAAAB4IIAAAAAAAAAAAAAAAAAAAAAAA=
334 TlRMTVNTUAACAAAACgAKADgAAAAFgooCBqqVKFrKPCMAAAAAAAAAAEgASABCAAAABgOAJQAAAA9JAEkAUwAwADEAAgAKAEkASQBTADAAMQABAAoASQBJAFMAMAAxAAQACgBJAEkAUwAwADEAAwAKAEkASQBTADAAMQAHAAgAHwMI0VPy1QEAAAAA
```

#### [smtp-ntlm-info](https://nmap.org/nsedoc/scripts/smtp-ntlm-info.html) NSE Script

```sh
nmap -p 587 --script smtp-ntlm-info --script-args smtp-ntlm-info.domain={{< param "war.rdomain" >}} {{< param "war.rhost" >}}
```

## Commands

```txt
HELO        Identify to the SMTP server.
EHLO        Alternative HELO for Extended SMTP protocol.
MAIL FROM:  Sender's email address.
RCPT TO:    Recipient's email address.
DATA        Initiate message content transfer. Command is terminated with a line containing only a .
RSET        Reset the session. Connection will not be closed.
VRFY        Verify username or mailbox.
NOOP        No-op. Keeps connection open.
QUIT        Ends session.
```

{{<note>}}
Sessions must start with HELO and end with QUIT.
{{</note>}}

## SMTP Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

## Configuration files

```
sendmail.cf
submit.cf
```

[^rfc-5321]: “RFC 5321 - Simple Mail Transfer Protocol.” IETF Tools, https://tools.ietf.org/html/rfc5321..
[^openssl]: OpenSSL Foundation, Inc. “/Docs/Manmaster/Man1/Openssl.Html.” OpenSSL.Org, https://www.openssl.org/docs/manmaster/man1/openssl.html.
[^nse-smtp-nltm-info]: “Smtp-Ntlm-Info NSE Script.” Nmap: The Network Mapper - Free Security Scanner, https://nmap.org/nsedoc/scripts/smtp-ntlm-info.html.
[^ntlm-disclosure]: m8r0wn. “Internal Information Disclosure Using Hidden NTLM Authentication | by M8r0wn | Medium.” Medium, Medium, 9 Mar. 2020, https://medium.com/@m8r0wn/internal-information-disclosure-using-hidden-ntlm-authentication-18de17675666.
