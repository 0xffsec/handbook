---
title: HTTP/S - 80 / 443
description: "HTTP/S (Hypertext Transfer Protocol Secure) Enumeration for Pentesting"
weight: 80
---
# HTTP/S

{{<hint info>}}
Hypertext Transfer Protocol and Hypertext Transfer Protocol Secure.

**Default Ports**
- HTTP: 80
- HTTPS (HTTP over TLS or SSL): 443
{{</hint>}}

## Directory Enumeration

As a first step, while we browse the web/application, it is a good idea to do some files and directories enumeration, look for **unlinked content**, **temporary directories**, and **backups**.
Widely used tools include [dirbuster](https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project), [gobuster](https://github.com/OJ/gobuster), [dirb](https://sourceforge.net/projects/dirb/) and the suite [Burp](https://portswigger.net/burp).

### Wordlists

Included in [Kali's wordlists package](https://tools.kali.org/password-attacks/wordlists).

- James Fisher's 2.3 medium ( 1.9M - 220560 lines )

    `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

- James Fisher's 2.3 small ( 709K - 87664 lines )

    `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt`

- dirb common ( 36K - 4614 lines )

    `/usr/share/wordlists/dirb/common.txt`

- dirb big ( 180K - 20469 lines )

    `/usr/share/wordlists/dirb/big.txt`

Not included in Kali.
- [Jhaddix's](https://twitter.com/Jhaddix) `content_discovery_all` ( 5.9M - 373535 lines )

    <https://gist.github.com/jhaddix/b80ea67d85c13206125806f0828f4d10>

- Daniel Miessler's [Robots Disallowed](https://github.com/danielmiessler/RobotsDisallowed) and [SecLists](https://github.com/danielmiessler/SecLists/) (Includes the former one).

    [https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content]()

### gobuster [^gobuster]

```sh
gobuster dir -t 30 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://{{< param "war.rhost" >}}/
```
{{<details "Parameters">}}
- `dir`: directory brute-forcing mode.
- `-t <n>`: number of concurrent threads (default 10).
- `-w <wordlist>`: path to the wordlist.
- `-u <URL>`: target URL.
{{</details>}}

{{<hint info>}}
- Iterate over the results.
- Include status code 403 (Forbidden Error) and brutefoce these directories.
- Add more file extensions to search for; In `gobuster`: `-x sh,pl`.
{{</hint>}}

## Source Code

### Inspect

It is a good habit to take a quick look at the pages' source code, scripts, and console outputs.

To active `View Source`, context-click on the page and select `View Page Source` or with the `Ctrl+U` or `Cmd+U` shortcut.

{{<hint info>}}
Many browsers include a powerful suite of tools, also known as devtools, to inspect and interact with the target website.
{{</hint>}}

### Download

If the target uses an open-source app, downloading its codebase will provide helpful information about configuration files, open resources, **default credentials**, etc.

[^gobuster]: Reeves, OJ. “GitHub - OJ/Gobuster.” GitHub, https://github.com/OJ/gobuster.
[^dirb]: Pinuaga, Ramon. “DIRB .” DIRB Homepage, http://dirb.sourceforge.net/.
