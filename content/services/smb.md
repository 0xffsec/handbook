---
menu: "SMB - 139 / 445"
title: "SMB (Server Message Block) Service Enumeration"
weight: 139
---
# SMB (Server Message Block)

## At a Glance

{{<highlight>}}
**Default Ports**
- SMB over NBT ([NetBIOS]({{< ref "netbios" >}}) over TCP/IP): 139
- SMB over TCP/IP: 445
{{</highlight>}}

SMB is a network communication protocol for providing shared access to files, printers, and serial ports between nodes on a network. It also provides an authenticated IPC (inter-process communication) mechanism.[^wiki-smb]

#### Windows SMB Ports and Protocols

Originally,
in Windows NT,
SMB ran on top of NBT (NetBIOS over TCP/IP),
which uses ports UDP 137 and 138,
and TCP 139.
With Windows 2000,
was introduced what Microsoft calls "direct hosting",
the option to run "NetBIOS-less" SMB,
directly over TCP/445.

Older versions of Windows
(with NBT enabled)
will try to connect to both port 139
and 445 simultaneously,
while in newer versions,
port 139 is a fall-back port,
as clients will try to connect to port 445
by default.[^vidstrom-smb-ports]

## SMB Host Discovery

Refer to [host discovert with nbtscan]({{< ref "host-discovery#nbtscan---netbios-nbtscan" >}}).

## Server Version

#### Metasploit SMB Auxiliary Module [^metasploit-smb-auxiliary-module]

```sh
msf> use auxiliary/scanner/smb/smb_version
msf> set rhost {{< param "war.rhost" >}}
msf> run
```

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

## Enumeration

{{<note>}}
If `Protocol negotiation failed: NT_STATUS_IO_TIMEOUT` is returned,
refer to [SMB Protocol Negotiation Failed]({{< ref "smb-protocol-negotiation-failed" >}})
{{</note>}}

#### enum4linux [^enum4linux]

```sh
enum4linux -a {{< param "war.rhost" >}}
```

With credentials:
```sh
enum4linux -a -u "<username>" -p "<passwd>" {{< param "war.rhost" >}}
```

{{<details "Parameters">}}
- `-a`: Do all simple enumeration (-U -S -G -P -r -o -n -i).
- `-u <user>`: specify username to use.
- `-p <pass>`: specify password to use.
{{</details>}}

#### NSE Scripts

```sh
nmap --script "safe or smb-enum-*" -p 139,445 {{< param "war.rhost" >}}
```

{{<note>}}
NSE SMB enumeration scripts:
- `smb-enum-domains`
- `smb-enum-groups`
- `smb-enum-processes`
- `smb-enum-services`
- `smb-enum-sessions`
- `smb-enum-shares`
- `smb-enum-users`
{{</note>}}

#### smbclient [^smbclient]

List available shares.

```sh
smbclient -N -L //{{< param "war.rhost" >}}
```

Connect to a share.

```sh
smbclient -N //{{< param "war.rhost" >}}/Share
```

{{<details "Parameters">}}
- `-N`: remove the password prompt from the client to the user.
- `-L`: list services available on the server.
{{</details>}}

## RPC Enumeration

{{<note>}}
`rpcclient`, `impacket`, and more, under [RPC Enumeration]({{< ref "msrpc#enumeration" >}}).
{{</note>}}

## Null Session

### Windows Administrative Shares

Administrative shares are hidden shares that provide administrators the ability to remotely manage hosts. **They are automatically created and enabled by default**.

{{<note>}}
It is worth clarifying these shares are not hidden but removed from views just by appending a dollar sign (`$`) to the share name. Ultimately, the share will be part of the result if listing from a Unix-based system or by using: `net share` and `net view /all`.
{{</note>}}

Various shares are exposed to clients via SMB, as follows:
- `C$`: C Drive on the remote machine.
- `Admin$`: Windows installation directory.
- `IPC$`: The inter-process communication or IPC share.
- `SYSVOL` and `NETLOGON`: domain controller shares.
- `PRINT$` and `FAX$`: printer and fax shares.

### IPC$ Share

`IPC$` is a special share
used to facilitate inter-process communication (`IPC`).
It does not allow access to files or directories,
but it allows to communicate
with processes running on the remote system.

Specifically, **`IPC$`, exposes named pipes**,
which can be written or read
to communicate with remote processes.
These named pipes
are opened by the application
and registered with SMB
so that it can be exposed by the `IPC$` share.

They are usually used
to perform specific functions on the remote system,
also known as **RPC** or remote procedure calls.

Some versions of Windows
allow you to authenticate
and mount the `IPC$` share
without providing a username and password.
Such a connection is often called a NULL session,
which,
despite its limited privileges,
could be used to make multiple RPC calls
and obtain useful information
about the remote system.[^sensepost-ipc]

{{<note>}}
RPC endpoints exposed via IPC$
include the Server service,
Task Scheduler,
Local Security Authority (LSA),
and Service Control Manager (SCM).
Upon authenticating,
you can use these
to enumerate user and system details,
access the registry,
and execute commands
{{</note>}}

In Linux
[enum4linux]({{< ref "#enum4linux-enum4linux" >}}) utility
can be used to dump data
from these service

Refer to [MSRPC]({{< ref "msrpc" >}}) for more about RPC.

## Mount Shares [^mount-smb]

```sh
mount -t cifs -o username=user,password=password //{{< param "war.rhost" >}}/Share /mnt/share
```

## Download Files

Create a tar file of the files beneath `users/docs`. [^smbclient]

```sh
smbclient //{{< param "war.rhost" >}}/Share "" -N -Tc backup.tar users/docs
```

{{<details "Parameters">}}
- `-N`: remove the password prompt from the client to the user.
- `-T`: TAR options.
- `c`: Create a tar backup archive on the local system.
{{</details>}}

## Brute Forcing

Refer to [SMB Brute Forcing]({{< ref "brute-forcing#smb">}})

## SMB Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})


[^wiki-smb]: Contributors to Wikimedia projects. “Server Message Block - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 26 Oct. 2003, https://en.wikipedia.org/wiki/Server_Message_Block.
[^vidstrom-smb-ports]: “The Use of TCP Ports 139 and 445 in Windows.” Vidstrom Labs, https://vidstromlabs.com/blog/the-use-of-tcp-ports-139-and-445-in-windows/.
[^metasploit-smb-auxiliary-module]: “Scanner SMB Auxiliary Modules - Metasploit Unleashed.” Infosec Training and Penetration Testing | Offensive Security, https://www.offensive-security.com/metasploit-unleashed/scanner-smb-auxiliary-modules/.
[^mcnab-nsa]: McNab, Chris. Network Security Assessment. “O’Reilly Media, Inc.,” 2007, p. 281.
[^smbclient]: “Smbclient.” Samba - Opening Windows to a Wider World, https://www.samba.org/samba/docs/current/man-html/smbclient.1.html.
[^sensepost-ipc]: “A New Look at Null Sessions and User Enumeration.” SensePost, https://sensepost.com/blog/2018/a-new-look-at-null-sessions-and-user-enumeration/.
[^mount-smb]: “Mounting Samba Shares from a Unix Client.” SambaWiki, https://wiki.samba.org/index.php/Mounting_samba_shares_from_a_unix_client.
[^enum4linux]: “Enum4linux.” Enum4linux | Portcullis Labs, Portcullis Computer Security Ltd & Portcullis Inc., 16 Sept. 2008, https://labs.portcullis.co.uk/tools/enum4linux/.
