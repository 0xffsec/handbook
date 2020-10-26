---
menu: Upgrade to Full TTY
title: Upgrade Simple Shells to Fully Interactive TTYs
weight: 402
---

# Upgrade to Fully Interactive TTYs

## At a Glance

More often than not, reverse, or bind shells are shells with limited interactive capabilities. Meaning no job control, no auto-completion, no `STDERR` output, and, most important, poor signal handling and limited commands support. To fully leverage the shell it is convenient to upgrade to an interactive TTY with extended features.

{{<note>}}
To check if the shell is a TTY shell use the `tty` command.
{{</note>}}

### Shell to Bash

Upgrade from shell to bash.

```sh
SHELL=/bin/bash script -q /dev/null
```

### Python PTY Module [^python-pty-module]

Spawn `/bin/bash` using [Python's PTY module](https://docs.python.org/3/library/pty.html), and connect the controlling shell with its standard I/O.

```sh
python -c 'import pty; pty.spawn("/bin/bash")'
```

### Fully Interactive TTY

Background the current remote shell (`^Z`), update the **local** terminal line settings with `stty`[^stty] and bring the remote shell back.

```sh
stty raw -echo && fg
```

After bringing the job back the cursor will be pushed to the left. Reinitialize the terminal with `reset`.

[^python-pty-module]: “Pty — Pseudo-Terminal Utilities.” 3.8.3 Documentation, https://docs.python.org/3/library/pty.html.
[^stty]: “Stty.” Linux Manual Page, https://man7.org/linux/man-pages/man1/stty.1.html.
