---
title: SSH Legacy Key Exchange
bookToc: false
---

# SSH Legacy Key Exchange Algorithm[^ssh-legacy]

When an SSH client connects to a server, each side offers sets of connection parameters to the other. For a successful connection, there must be at least one mutually compatible set for each parameter.  
If the client and the server cannot agree on a mutual set, in this case, the key exchange algorithm, the connection will fail and OpenSSH will return an error message like this:

```sh
Unable to negotiate with 10.10.10.7 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
```

The server offered `diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1`. OpenSSH supports these methods but does not enable it by default because is weak and within the theoretical range of the so-called Logjam attack.  

More about the Logjam attack in [Imperfect Forward Secrecy:How Diffie-Hellman Fails in Practice](https://weakdh.org/).

## Solution

**If upgrading is not immediately possible** you can re-enable the algorithms either on the command-line:

```sh
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.10.7
```

or in the `~/.ssh/config` file:

```sh
Host 10.10.10.7
    KexAlgorithms +diffie-hellman-group1-sha1
```

[^ssh-legacy]: “OpenSSH: Legacy Options.” OpenSSH, https://www.openssh.com/legacy.html.
