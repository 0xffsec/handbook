---
menu: "MSRPC - 135 / 593"
title: "MSRPC (Microsoft Remote Procedure Call) Service Enumeration"
weight: 135
---
# MSRPC (Microsoft Remote Procedure Call)

## At a Glance

{{<highlight>}}
**Default Ports**:
- RPC Endpoint Mapper: 135
- HTTP: 593
{{</highlight>}}

MSRPC is an interprocess communication (IPC) mechanism
that allows client/server software communcation.
That process can be on the same computer,
on the local network (LAN),
or across the Internet.
Its purpose is to
provide a common interface
between applications.

Within Windows environments,
many server applications
are exposed via RPC.

The Microsoft RPC mechanism
uses other IPC mechanisms,
such as named pipes,
[NetBIOS]({{< ref "netbios" >}})
or Winsock,
to establish communications
between the client and the server.
Along with `IPC$` transport,
TCP, UDP, and HTTP
are used to provide access to services

{{< figure src="/images/msrpc.png" alt="MSRPC" title="Source: Network Security Assesment 3rd Edition." >}}

The RPC locator service
works much like the RPC portmapper service
found in Unix environments.[^ms-rpc]

## Enumeration

You can query the RPC locator service
and individual RPC endpoints
to catalog services running over
TCP, UDP, HTTP, and SMB (via named pipes).

Each returned IFID value
represents an RPC service.
See [Notable RPC Interfaces](#notable-rpc-interfaces-oreilly-net-assesment).

By default, `impacket`
will try to match them
with a list of well known endpoints.
[^oreilly-net-assesment]

#### impacket `pcdump.py` [^impacket]

Dump the list of RPC endpoints.

```sh
rpcdump.py {{< param "war.rhost" >}}
```
{{<details "Parameters">}}
- `target`: `[[domain/]username[:password]@]address`
- `-port <ports>`: Destination port to connect to SMB server. Default: 135.
{{</details>}}

#### impacket `samrdump.py` [^impacket]

List system user accounts,
available resource shares
and other sensitive information
exported through the SAMR (Security Account Manager Remote) interface.

```sh
samrdump.py {{< param "war.rhost" >}}
```
{{<details "Parameters">}}
- `target`: `[[domain/]username[:password]@]address`
- `-port <ports>`: Destination port to connect to SMB server. Default: 445.
{{</details>}}

#### [msrpc-enum](https://nmap.org/nsedoc/scripts/msrpc-enum.html) NSE Script
```sh
nmap -sV -script msrpc-enum -Pn {{< param "war.rhost" >}}
```

## Query RPC

The `rpcclient` can be used to interact with
individual RPC endpoints via named pipes.
By default,
Windows systems and Windows 2003 domain controllers
allow anonymous ([Null Sessions]({{< ref "smb#null-session">}})) access to SMB,
so these interfaces
can be queried in this way.

{{<note>}}
If null session access
is not permitted,
a valid username and password must be provided.
{{</note>}}

#### rpcclient [^rpcclient]
```sh
rpcclient -U "" -N {{< param "war.rhost" >}}
```
{{<details "Parameters">}}
- `-U`: Set the network username.
- `-N`: Don't ask for a password.
{{</details>}}

Commands that you can issue to SAMR, LSARPC, and LSARPC-DS.[^oreilly-net-assesment]

| Command               | Interface | Description                                   |
|-----------------------|-----------|-----------------------------------------------|
| `queryuser`           | SAMR      | Retrieve user information.                    |
| `querygroup`          | SAMR      | Retrieve group information.                   |
| `querydominfo`        | SAMR      | Retrieve domain information.                  |
| `enumdomusers`        | SAMR      | Enumerate domain users.                       |
| `enumdomgroups`       | SAMR      | Enumerate domain groups.                      |
| `createdomuser`       | SAMR      | Create a domain user.                         |
| `deletedomuser`       | SAMR      | Delete a domain user.                         |
| `lookupnames`         | LSARPC    | Look up usernames to SID values.              |
| `lookupsids`          | LSARPC    | Look up SIDs to usernames (RID cycling).      |
| `lsaaddacctrights`    | LSARPC    | Add rights to a user account.                 |
| `lsaremoveacctrights` | LSARPC    | Remove rights from a user account.            |
| `dsroledominfo`       | LSARPC-DS | Get primary domain information.               |
| `dsenumdomtrusts`     | LSARPC-DS | Enumerate trusted domains within an AD forest |

## Notable RPC Interfaces [^oreilly-net-assesment]

**IFID**: 12345778-1234-abcd-ef00-0123456789ab\
**Named Pipe**: `\pipe\lsarpc`\
**Description**: LSA interface, used to enumerate users.

**IFID**: 3919286a-b10c-11d0-9ba8-00c04fd92ef5\
**Named Pipe**: `\pipe\lsarpc`\
**Description**: LSA Directory Services (DS) interface, used to enumerate domains and trust relationships.

**IFID**: 12345778-1234-abcd-ef00-0123456789ac\
**Named Pipe**: `\pipe\samr`\
**Description**: LSA SAMR interface, used to access public SAM database elements (e.g., usernames) and brute-force user passwords regardless of account lockout policy.

**IFID**: 1ff70682-0a51-30e8-076d-740be8cee98b\
**Named Pipe**: `\pipe\atsvc`\
**Description**: Task scheduler, used to remotely execute commands.


**IFID**: 338cd001-2244-31f1-aaaa-900038001003\
**Named Pipe**: `\pipe\winreg`\
**Description**: Remote registry service, used to access and modify the system registry.


**IFID**: 367abb81-9844-35f1-ad32-98f038001003\
**Named Pipe**: `\pipe\svcctl`\
**Description**: Service control manager and server services, used to remotely start and stop services and execute commands.


**IFID**: 4b324fc8-1670-01d3-1278-5a47bf6ee188\
**Named Pipe**: `\pipe\srvsvc`\
**Description**: Service control manager and server services, used to remotely start and stop services and execute commands.


**IFID**: 4d9f4ab8-7d1c-11cf-861e-0020af6e7c57\
**Named Pipe**: `\pipe\epmapper`\
**Description**: DCOM interface, used for brute-force password grinding and information gathering via WM.

## Further Reading

- [Impacket | SecureAuth](https://www.secureauth.com/labs/open-source-tools/impacket)

[^ms-rpc]: “How RPC Works: Remote Procedure Call (RPC).” Technical Documentation, API, and Code Examples | Microsoft Docs, https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc738291(v=ws.10)?redirectedfrom=MSDN.
[^oreilly-net-assesment]: McNab, Chris. Network Security Assessment, 3rd Edition. O’Reilly Media, Inc., 2016.
[^impacket]: “Impacket | SecureAuth.” SecureAuth, https://www.secureauth.com/labs/open-source-tools/impacket.
[^rpcclient]: “Rpcclient(1) - Linux Man Page.” Linux Documentation, https://linux.die.net/man/1/rpcclient.
