---
menu: "NetBIOS - 137 / 138 / 139"
title: "NetBIOS (Network Basic Input/Output System) Service Enumeration"
weight: 137
---
# NetBIOS (Network Basic Input/Output System)

## At a Glance

{{<highlight>}}
**Default Port/s**:
- NetBIOS Name Service: UDP/137
- NetBIOS Datagram Service: UDP/138
- NetBIOS Session Service: TCP/139
{{</highlight>}}

NetBIOS is a non-routable service
that allows applications and computers
to communicate over a local area network (LAN).

As an API,
NetBIOS relies on network protocols to communicate.
In modern networks,
NetBIOS runs over TCP/IP via the NetBIOS over TCP/IP protocol
or NBT.[^wiki-netbios]

NetBIOS provides three distinct services:
- Name Service (NetBIOS-NS) for name registration and resolution.
- Datagram Distribution Service (NetBIOS-DGM) for connectionless communication.
- Session Service (NetBIOS-SSN) for connection-oriented communication.

{{<note>}}
[SMB]({{< ref "smb" >}}) runs **on top** of the Session Service and Datagram Service.
It is not an integral part of NetBIOS.
{{</note>}}

#### Name Service

On a NetBIOS network,
applications locate and identify each other
through their NetBIOS names.
NetBIOS Name Service serves much the same purpose as DNS does:
translate human-readable names to IP addresses.

NetBIOS names are 16 octets long,
however,
Microsoft limits the hostname to 15 characters
and reserves the 16th character
as a NetBIOS Suffix.
This suffix,
aka NetBIOS End Character (endchar),
describes the service or name record type.[^ms-names]
See [NetBIOS Suffixes](#netbios-suffixes).

In NBT,
NetBIOS-NS runs on UDP port 137.

#### Datagram Distribution Service

The Datagram service is an unreliable,
non-sequenced,
connectionless service.

Datagrams may be sent to a specific name
or explicitly broadcast.
Usually used to broadcast names
and register services.[^rfc-1001]

In NBT,
NetBIOS-DGM runs on UDP port 138.

#### Session Service

The Session service offers a reliable message exchange,
conducted between a pair of NetBIOS applications.
Sessions are full-duplex,
sequenced,
and reliable.[^rfc-1001]

The service facilitates authentication
and provides access to shared resources,
such as files and printers.
It is where [NULL Sessions]({{< ref "smb#null-session" >}}) are established.

In NBT,
NetBIOS-DGM runs on UDP port 139.

## Enumeration

#### nmblookup [^nmblookup]
```sh
nmblookup -A {{< param "war.rhost" >}}
```
{{<details "Parameters">}}
- `<name>`: NetBIOS Name
- `-A <ip>`: Interpret name as an IP Address and do a node status query on this address.
{{</details>}}

#### nbtscan [^nbtscan]
```sh
nbtscan {{< param "war.rhost" >}}
```

{{<note>}}
Continue NetBIOS enumeration with [SMB]({{< ref "smb" >}}).
{{</note>}}

## NetBIOS Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

## NetBIOS Suffixes [^mcnab-nsa]

| Name                 | Suffix | Type | Service                                |
| -------------------- | ------ | ---- | -------------------------------------- |
| `<computername>`     | 00     | U    | Workstation                            |
| `<computername>`     | 01     | U    | Messenger                              |
| `<\\−−__MSBROWSE__>` | 01     | G    | Master Browser                         |
| `<computername>`     | 03     | U    | Messenger                              |
| `<computername>`     | 06     | U    | RAS Server                             |
| `<computername>`     | 1F     | U    | NetDDE                                 |
| `<computername>`     | 20     | U    | File Server                            |
| `<computername>`     | 21     | U    | RAS Client                             |
| `<computername>`     | 22     | U    | Microsoft Exchange Interchange         |
| `<computername>`     | 23     | U    | Microsoft Exchange Store               |
| `<computername>`     | 24     | U    | Microsoft Exchange Directory           |
| `<computername>`     | 30     | U    | Modem Sharing Server                   |
| `<computername>`     | 31     | U    | Modem Sharing Client                   |
| `<computername>`     | 43     | U    | SMS Clients Remote Control             |
| `<computername>`     | 44     | U    | SMS Administrators Remote Control Tool |
| `<computername>`     | 45     | U    | SMS Clients Remote Chat                |
| `<computername>`     | 46     | U    | SMS Clients Remote Transfer            |
| `<computername>`     | 4C     | U    | DEC Pathworks TCPIP                    |
| `<computername>`     | 42     | U    | McAfee Antivirus                       |
| `<computername>`     | 52     | U    | DEC Pathworks TCPIP                    |
| `<computername>`     | 87     | U    | Microsoft Exchange MTA                 |
| `<computername>`     | 6A     | U    | Microsoft Exchange IMC                 |
| `<computername>`     | BE     | U    | Network Monitor Agent                  |
| `<username>`         | BF     | U    | Network Monitor Application            |
| `<domain>`           | 03     | U    | Messenger                              |
| `<domain>`           | 00     | G    | Domain Name                            |
| `<domain>`           | 1B     | U    | Domain Master Browser                  |
| `<domain>`           | 1C     | G    | Domain Controllers                     |
| `<domain>`           | 1D     | U    | Master Browser                         |
| `<domain>`           | 1E     | G    | Browser Service Elections              |
| `<INet~Services>`    | 1C     | G    | IIS                                    |
| `<IS~computername>`  | 00     | U    | IIS                                    |
| `<computername>`     | [2B]   | U    | IBM Lotus Notes Server                 |
| `IRISMULTICAST`      | [2F]   | G    | IBM Lotus Notes                        |
| `IRISNAMESERVER`     | [33]   | G    | IBM Lotus Notes                        |
| `Forte_$ND800ZA`     | [20]   | U    | DCA IrmaLan Gateway Server Service     |

## Further Reading

- [Windows File Sharing: Facing the Mystery by Daniel Miessler](https://danielmiessler.com/blog/windowsfilesharing/)

[^wiki-netbios]: Contributors to Wikimedia projects. “NetBIOS - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 2 Feb. 2003, https://en.wikipedia.org/wiki/NetBIOS.
[^ms-names]: Deland-Han. “Name Computers, Domains, Sites, and OUs - Windows Server | Microsoft Docs.” Technical Documentation, API, and Code Examples | Microsoft Docs, https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/naming-conventions-for-computer-domain-site-ou.
[^mcnab-nsa]: McNab, Chris. Network Security Assessment. “O’Reilly Media, Inc.,” 2007, p. 195.
[^rfc-1001]: “RFC 1001 - Protocol Standard for a NetBIOS Service on a TCP/UDP Transport: Concepts and Methods.” IETF Tools, https://tools.ietf.org/html/rfc1001.
[^nmblookup]: “Nmblookup.” Samba - Opening Windows to a Wider World, https://www.samba.org/samba/docs/current/man-html/nmblookup.1.html.
[^nbtscan]: “Nbtscan - NETBIOS Nameserver Scanner.” Steve Friedl’s Home Page, http://unixwiz.net/tools/nbtscan.html. Accessed 28 Sept.
