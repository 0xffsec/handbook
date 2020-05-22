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
msf> set rhost {{< param rhost >}}
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

## List

Look for available shares using default credentials.

### smbclient [^smbclient]
```sh
smbclient -N -L //{{< param rhost >}}
```
{{<details "Parameters">}}
- `-N`: remove the password prompt from the client to the user.
- `-L`: list services available on the server.
{{</details>}}

## Connect

### smbclient [^smbclient]
```sh
smbclient -N //{{< param rhost >}}/Share
```

## Mount [^mount-smb]

```sh
mount -t cifs -o username=user,password=password //{{< param rhost >}}/Share /mnt/share
```

## Null Session Enumeration

### Windows Administrative Shares

Administrative shares are hidden shares that provide administrators the ability to remotely manage hosts. **They are automatically created and enabled by default**.  

{{<hint info>}}
It is worth clarifying these shares are not hidden but removed from views just by appending a dollar sign (`$`) to the share name. Ultimately, the share will be part of the result if listing from a Unix-based system or by using: `net share` and `net view /all`.
{{</hint>}}

Three common shares are:
- `C$`: allows access to the C Drive on the remote machine. 
- `Admin$`: allows access to the Windows installation directory.
- `IPC$`: allows inter-process communication or IPC.

### IPC$ Share

`IPC$` is a special share used to facilitate inter-process communication (`IPC`). In other words, it does not allow access to files or directories, but it allows to communicate with processes running on the remote system.   
Specifically, **`IPC$` exposes named pipes**, which can be written or read to communicate with remote processes. These named pipes are created when an application opens a pipe and registers it with SMB so that it can be exposed by the `IPC$` share.  
On the one hand, data written to the named pipe is sent to the remote process, and on the other hand, a local application reads the output information written by the remote process. They are usually used to perform specific functions on the remote system, also known as **RPC** or remote procedure calls.  

Some versions of Windows allow you to authenticate and mount the `IPC$` share without providing a username and password. Such a connection is often called a NULL session, which, despite its limited privileges, could be used to make multiple RPC calls and obtain useful information about the remote system.[^sensepost-ipc]


# Reference

[^wiki-smb]: Contributors to Wikimedia projects. “Server Message Block - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 26 Oct. 2003, https://en.wikipedia.org/wiki/Server_Message_Block.
[^vidstrom-smb-ports]: “The Use of TCP Ports 139 and 445 in Windows.” Vidstrom Labs, https://vidstromlabs.com/blog/the-use-of-tcp-ports-139-and-445-in-windows/.
[^metasploit-smb-auxiliary-module]: “Scanner SMB Auxiliary Modules - Metasploit Unleashed.” Infosec Training and Penetration Testing | Offensive Security, https://www.offensive-security.com/metasploit-unleashed/scanner-smb-auxiliary-modules/.
[^mcnab-nsa]: McNab, Chris. Network Security Assessment. “O’Reilly Media, Inc.,” 2007, p. 281.
[^smbclient]: “Smbclient.” Samba - Opening Windows to a Wider World, https://www.samba.org/samba/docs/current/man-html/smbclient.1.html.
[^sensepost-ipc]: “A New Look at Null Sessions and User Enumeration.” SensePost, https://sensepost.com/blog/2018/a-new-look-at-null-sessions-and-user-enumeration/.
[^mount-smb]: “Mounting Samba Shares from a Unix Client.” SambaWiki, https://wiki.samba.org/index.php/Mounting_samba_shares_from_a_unix_client.
