---
title: HTTP/S - 80 / 443
description: "HTTP/S (Hypertext Transfer Protocol Secure) Enumeration for Pentesting"
weight: 80

service: HTTP/S
service_description: Hypertext Transfer Protocol and Hypertext Transfer Protocol Secure.
service_port: 80 / 443
---
## Directory Enumeration

As a first step, while we browse the web/application, it is a good idea to do some files and directories enumeration, look for **unlinked content**, **temporary directories**, and **backups**.  
Widely used tools include [dirbuster](https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project), [gobuster](https://github.com/OJ/gobuster), [dirb](https://sourceforge.net/projects/dirb/) and the suite [Burp](https://portswigger.net/burp).

### Wordlists

Included in [Kali's wordlists package](https://tools.kali.org/password-attacks/wordlists).

- dirbuster's medium 2.3.  
`/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
- dirb common.  
`/usr/share/wordlists/dirb/common.txt`
- dirb big.  
`/usr/share/wordlists/dirb/common.txt`

Not included in Kali.
- [Jhaddix's](https://twitter.com/Jhaddix) `content_discovery_all`.  
<https://gist.github.com/jhaddix/b80ea67d85c13206125806f0828f4d10>

- [Daniel Miessler's](https://twitter.com/danielmiessler) [Robots Disallowed](https://github.com/danielmiessler/RobotsDisallowed) and [SecLists](https://github.com/danielmiessler/SecLists/) (Includes the former one).  
`/usr/share/seclists/Discovery/Web-Content/common.txt`  
`/usr/share/seclists/Discovery/Web-Content/RobotsDisallowed-Top1000.txt`

### gobuster [^gobuster]
```sh
gobuster dir -t 30 -k -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://10.0.0.3/ 
```
{{<details "Parameters">}}
- `dir`: directory brute-forcing mode.
- `-t <n>`: number of concurrent threads (default 10).
- `-k`: skip SSL certificate verification.
- `-w <wordlist>`: path to the wordlist.
- `-u <URL>`: target URL.
{{</details>}}

### dirb [^dirb]
```sh
dirb https://10.0.0.3/ /usr/share/seclists/Discovery/Web-Content/common.txt
```
## Reference

[^gobuster]: Reeves, OJ. “GitHub - OJ/Gobuster.” GitHub, https://github.com/OJ/gobuster.
[^dirb]: Pinuaga, Ramon. “DIRB .” DIRB Homepage, http://dirb.sourceforge.net/.
