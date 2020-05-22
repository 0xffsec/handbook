---
title: "FTP - 21"
description: "FTP (File Transfer Protocol) Enumeration for Pentesting"
weight: 21

service: FTP
service_description: File Transfer Protocol.
service_port: 21
---
FTP is a standard network protocol used for the transfer of files between a client and a server on a computer network.  
FTP is built on a client-server architecture using separate control and data connections between the client and the server. FTP authenticates users with a clear-text sign-in protocol, normally in the form of a username and password, but can connect anonymously if the server is configured to allow it. [^wiki-ftp]

## Banner Grabbing

#### Telnet

```sh
telnet {{< param rhost >}} 21
```

#### Netcat
```sh
nc -n {{< param rhost >}} 21
```

#### Nmap
```sh
nmap -sV -script=banner -p21 -Pn {{< param rhost >}}
```

#### FTP
```sh
ftp {{< param rhost >}}
```

## FTP Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

## Anonymous Login

{{<hint info>}}
During the [port scanning]({{< ref "port-scanning" >}}) phase Nmap's script scan (`-sC`), can be enabled to check for [FTP Bounce](https://nmap.org/nsedoc/scripts/ftp-bounce.html) and [Anonymous Login](https://nmap.org/nsedoc/scripts/ftp-anon.html).
{{</hint>}}

Try anonymous login using `anonymous:anonymous` credentials.

```sh
ftp {{< param rhost >}}
…
Name ({{< param rhost >}}:kali): anonymous
331 Please specify the password.
Password: [anonymous]
230 Login successful.
```

List **all** files in order.

```sh
ftp> ls -lat
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
…
226 Directory send OK.
```

## FTP Browser Client

{{<hint danger>}}
Due to its insecure nature, FTP support is being dropped by [Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1574475) and [Google Chrome](https://chromestatus.com/feature/6246151319715840).
{{</hint>}}

Try accessing `ftp://user:pass@{{< param rhost >}}` from your browser.  
If not credentials provided `anonymous:anonymous` is assumed.

## Brute Forcing

As an alternative to profiled passwords, [Daniel Miessler's SecLists](https://github.com/danielmiessler/SecLists) includes a handy list of [FTP default credentials](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt).  

### Hydra [^hydra]

Using a colon-separated `user:pass` list.

```sh
hydra -v -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt -f {{< param rhost >}} ftp
```

Using login user and passwords list.

```sh
hydra -v -l ftp -P /usr/share/wordlists/rockyou.txt.gz -f {{< param rhost >}} ftp
```

{{<details "Parameters">}}
- `-v`: verbose mode.
- `-C <user:pass file>`: colon-separated "login:pass" format.
- `-l <user>`: login with `user` name.
- `-P <passwords file>`: login with password from file.
- `-f`: exit after the first found user/password pair.
{{</details>}}

### Medusa [^medusa]

Medusa's combo files (colon-separated) should be in the format `host:username:password`. If any of the three values are missing, the respective information should be provided either as a global value or as a list in a file.

```sh
sed s/^/:/ /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt > /tmp/cplist.txt
medusa -C /tmp/cplist.txt -h {{< param rhost >}} -M ft
```

Using login user and passwords list.

```sh
medusa -u ftp -P /usr/share/wordlists/rockyou.txt -h {{< param rhost >}} -M ftp
```

{{<details "Parameters">}}
- `-u <user>`: login with `user` name.
- `-P <passwords file>`: login with password from file.
- `-h`: target hostname or IP address.
- `-M`: module to execute.
{{</details>}}


## Configuration files

Examine configuration files.[^0daysec-enum]

```
ftpusers
ftp.conf
proftpd.conf
```

## Miscellaneous

### Binary and ASCII

Binary and ASCII files have to be uploading using the `binary` or `ascii` mode respectively, otherwise, the file will become corrupted. Use the corresponding command to switch between modes.[^w3c-ftp-transfer]

### Recursively Download

Recursively download FTP folder content.[^so-ftp-mirroring]

```sh
wget -m ftp://user:pass@{{< param rhost >}}/
```

# Reference

[^wiki-ftp]: Contributors to Wikimedia projects. “File Transfer Protocol - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 24 May 2002, https://en.wikipedia.org/wiki/File_Transfer_Protocol.
[^so-ftp-mirroring]: Thibaut Barrère. “Command Line - How to Recursively Download a Folder via FTP on Linux - Stack Overflow.” Stack Overflow, https://stackoverflow.com/a/113900/578050.
[^0daysec-enum]: “Penetration Testing Methodology” 0DAYsecurity.Com, http://www.0daysecurity.com/penetration-testing/enumeration.html.
[^w3c-ftp-transfer]: “RFC959: FTP: Data Transfer Functions.” World Wide Web Consortium (W3C), https://www.w3.org/Protocols/rfc959/3_DataTransfer.html.
[^hydra]: Heuse, Marc. “GitHub - Vanhauser-Thc/Thc-Hydra: Hydra.” GitHub, https://github.com/vanhauser-thc/thc-hydra. Accessed 12 May 2020.
[^medusa]: “Foofus Networking Services - Medusa.” Foofus.Net | Foofus.Net Advanced Security Services Forum, http://foofus.net/goons/jmk/medusa/medusa.html.
