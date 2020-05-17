---
title: "SMB - 139 / 445"
description: "SMB (Server Message Block) Enumeration for Pentesting"
weight: 139

service: SMB
service_description: Server Message Block.
service_port: 139 / 445
---
SMB is a network communication protocol for providing shared access to files, printers, and serial ports between nodes on a network. It also provides an authenticated IPC (inter-process communication) mechanism.[^wiki-smb]

#### Windows SMB Ports and Protocols

{{<hint info>}}
**TL;DR**
- Port 139: SMB over NBT (NetBIOS over TCP/IP).
- port 445: SMB directly over TCP/IP.
{{</hint>}}

Originally, in Windows NT, SMB ran on top of NBT (NetBIOS over TCP/IP), which uses ports UDP 137 and 138, and TCP 139. With Windows 2000, was introduced the option to run "NetBIOS-less" SMB directly over TCP/IP, without the extra NBT layer on port 445.  
Older versions of Windows (with NBT enabled) will try to connect to both port 139 and 445 simultaneously, while in newer versions, port 139 is a fall-back port, as clients will try to connect to port 445 by default.[^vidstrom-smb-ports]

## SMB Host Discovery

Refer to [host discovert with nbtscan]({{< ref "host-discovery#nbtscan---netbios-nbtscan" >}}).

## Server Version

### Metasploit SMB Auxiliary Module [^metasploit-smb-auxiliary-module]

```sh
msf> use auxiliary/scanner/smb/smb_version
msf> set rhost 10.0.0.3
msf> run
```

## SMB Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

## Common Login Credentials

Backup and Management software requires dedicated user accounts on the server or local machine to function, and are often set with a weak password. [^mcnab-nsa]

| Usernmae | Password |
| -------- | -------- |
| (blank) | (blank) |
| `Administrator` `admin` `guest` | (blank) `admin` `password` |
| `arcserve` | `arcserve` `backup` |
| `tivoli` | `tivoli` |
| `backupexec` | `backupexec` `backup` |
| `test` | `test` |

## List Shares

Look for available shares using default credentials.

### smbclient [^smbclient]
```sh
smbclient --no-pass -L //10.0.0.3
```
{{<details "Parameters">}}
- `--no-pass`: remove the password prompt from the client to the user.
- `-L`: list services available on the server.
{{</details>}}

# Reference

[^wiki-smb]: Contributors to Wikimedia projects. “Server Message Block - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 26 Oct. 2003, https://en.wikipedia.org/wiki/Server_Message_Block.
[^vidstrom-smb-ports]: “The Use of TCP Ports 139 and 445 in Windows.” Vidstrom Labs, https://vidstromlabs.com/blog/the-use-of-tcp-ports-139-and-445-in-windows/.
[^metasploit-smb-auxiliary-module]: “Scanner SMB Auxiliary Modules - Metasploit Unleashed.” Infosec Training and Penetration Testing | Offensive Security, https://www.offensive-security.com/metasploit-unleashed/scanner-smb-auxiliary-modules/.
[^mcnab-nsa]: McNab, Chris. Network Security Assessment. “O’Reilly Media, Inc.,” 2007, p. 281.
[^smbclient]: “Smbclient.” Samba - Opening Windows to a Wider World, https://www.samba.org/samba/docs/current/man-html/smbclient.1.html.
