---
menu: "FTP - 21"
title: "FTP (File Transfer Protocol) Service Enumeration"
weight: 21
---
# FTP (File Transfer Protocol)

## At a Glance

{{<highlight>}}
**Default Port:** 21
{{</highlight>}}

FTP is a standard network protocol used for the transfer of files between a client and a server on a computer network.
FTP is built on a client-server architecture using separate control and data connections between the client and the server. FTP authenticates users with a clear-text sign-in protocol, normally in the form of a username and password, but can connect anonymously if the server is configured to allow it. [^wiki-ftp]

## Banner Grabbing

#### Telnet

```sh
telnet {{< param "war.rhost" >}} 21
```

#### Netcat
```sh
nc -n {{< param "war.rhost" >}} 21
```

#### [banner](https://nmap.org/nsedoc/scripts/banner.html) NSE Script
```sh
nmap -sV -script banner -p21 -Pn {{< param "war.rhost" >}}
```

#### FTP
```sh
ftp {{< param "war.rhost" >}}
```

## FTP Exploits Search

Refer to [Exploits Search]({{< ref "exploits-search">}})

## Anonymous Login

{{<note>}}
During the [port scanning]({{< ref "port-scanning" >}}) phase Nmap's script scan (`-sC`), can be enabled to check for [FTP Bounce](https://nmap.org/nsedoc/scripts/ftp-bounce.html) and [Anonymous Login](https://nmap.org/nsedoc/scripts/ftp-anon.html).
{{</note>}}

Try anonymous login using `anonymous:anonymous` credentials.

```sh
ftp {{< param "war.rhost" >}}
…
Name ({{< param "war.rhost" >}}:kali): anonymous
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

{{<note>}}
Due to its insecure nature, FTP support is being dropped by [Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1574475) and [Google Chrome](https://chromestatus.com/feature/6246151319715840).
{{</note>}}

Try accessing `ftp://user:pass@{{< param "war.rhost" >}}` from your browser.
If not credentials provided `anonymous:anonymous` is assumed.

## Brute Forcing

Refer to [FTP Brute Forcing]({{< ref "brute-forcing#ftp">}})

{{<note>}}
[SecLists](https://github.com/danielmiessler/SecLists) includes a handy list of [FTP default credentials](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt).
{{</note>}}

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
wget -m ftp://user:pass@{{< param "war.rhost" >}}/
```

[^wiki-ftp]: Contributors to Wikimedia projects. “File Transfer Protocol - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 24 May 2002, https://en.wikipedia.org/wiki/File_Transfer_Protocol.
[^so-ftp-mirroring]: Thibaut Barrère. “Command Line - How to Recursively Download a Folder via FTP on Linux - Stack Overflow.” Stack Overflow, https://stackoverflow.com/a/113900/578050.
[^0daysec-enum]: “Penetration Testing Methodology” 0DAYsecurity.Com, http://www.0daysecurity.com/penetration-testing/enumeration.html.
[^w3c-ftp-transfer]: “RFC959: FTP: Data Transfer Functions.” World Wide Web Consortium (W3C), https://www.w3.org/Protocols/rfc959/3_DataTransfer.html.
