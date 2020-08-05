---
title: Brute Forcing
weight: 201
---

# Brute Forcing

A brute-forcing attack consists of
systematically enumerating
all possible candidates for the solution
and checking whether each candidate
satisfies the problem's statement.

In cryptography,
a brute-force attack
involves systematically
checking all possible keys
until the correct key is found.
[^brute-force-wiki]


## Default Credentials

- [DefaultPassword](https://default-password.info/)
- [CIRT.net Password DB](https://www.cirt.net/passwords)
- [Default Router Passwords List](https://192-168-1-1ip.mobi/default-router-passwords-list/)

{{<hint info>}}
[SecLists](https://github.com/danielmiessler/SecLists) and [WordList Compendium](https://github.com/Dormidera/WordList-Compendium)
also include default passwords lists.
{{</hint>}}

## Wordlists

- [SecLists - The Pentester's Companion](https://github.com/danielmiessler/SecLists)
- [Probable Wordlists](https://github.com/berzerk0/Probable-Wordlists)
- [WordList Compendium](https://github.com/Dormidera/WordList-Compendium)
- [Jhaddix Content Discovery All](https://gist.github.com/jhaddix/b80ea67d85c13206125806f0828f4d10)
- [Google Fuzzing Forum](https://github.com/google/fuzzing)
- [CrackStation's Password Cracking Dictionary](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm)

## Wordlist Generation

### CeWL [^cewl]
```sh
cewl {{< param "war.rdomain" >}} -m 3 -w wordlist.txt
```
{{<details "Parameters">}}
- `-m <length>`: Minimum word length.
- `-w <file>`: Write the output to `<file>`.
{{</details>}}

### Crunch [^crunch]

#### Simple Wordlist

```sh
crunch 6 12 abcdefghijk1234567890\@\! -o wordlist.txt
```

#### String Permutation

```sh
crunch 1 1 -p target pass 2019 -o wordlist.txt
```

#### Patterns

```sh
crunch 9 9 0123456789 -t @target@@ -o wordlist.txt
```

{{<details "Parameters">}}
- `<min-len>`: The minimum string length.
- `<max-len>`: The maximum string length.
- `<charset>`: Characters set.
- `-o <file>`: Specifies the file to write the output to.
- `-p <charset or strings>`: Permutation.
- `-t <pattern>`:
    Specifies a pattern, eg: `@@pass@@@@`.
    - `@` will insert lower case characters
    - `,` will insert upper case characters
    - `%` will insert numbers
    - `^` will insert symbols
{{</details>}}

## Password Profiling

### CUPP [^cupp]
```sh
cupp -i
```
{{<details "Parameters">}}
- `-i`:  Interactive uestions for user password profiling.
{{</details>}}

## Word Mangling

### john [^john]
```sh
john --wordlist=wordlist.txt --rules --stdout
```
{{<details "Parameters">}}
- `--wordlist <file>`: Wordlist mode, read words from `<file>` or `stdin`.
- `--rules[:CustomRule]`: Enable word mangling rules. Use default or add `[:CustomRule]`.
- `--stdout`: Output candidate passwords.
{{</details>}}

{{<hint info>}}
Custom rules can be appended
to John's configuration file `john.conf`.

See: [KoreLogic's Word Mangling Rule](https://gist.github.com/maxrodrigo/b9ec4e4578a9f6968470381214f1a340)
{{</hint>}}

## Services

### FTP

#### Hydra [^hydra]

Using a colon-separated `user:pass` list.

```sh
hydra -v -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt -f {{< param "war.rhost" >}} ftp
```

Using login user and passwords list.

```sh
hydra -v -l ftp -P /usr/share/wordlists/rockyou.txt.gz -f {{< param "war.rhost" >}} ftp
```

{{<details "Parameters">}}
- `-v`: verbose mode.
- `-C <user:pass file>`: colon-separated "login:pass" format.
- `-l <user>`: login with `user` name.
- `-P <passwords file>`: login with password from file.
- `-f`: exit after the first found user/password pair.
{{</details>}}

#### Medusa [^medusa]

Medusa's combo files (colon-separated)
should be in the format `host:username:password`.
If any of the three values are missing,
the respective information
should be provided either
as a global value
or as a list in a file.

```sh
sed s/^/:/ /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt > /tmp/cplist.txt
medusa -C /tmp/cplist.txt -h {{< param "war.rhost" >}} -M ft
```

Using login user and passwords list.

```sh
medusa -u ftp -P /usr/share/wordlists/rockyou.txt -h {{< param "war.rhost" >}} -M ftp
```

{{<details "Parameters">}}
- `-u <user>`: login with `user` name.
- `-P <passwords file>`: login with password from file.
- `-h`: target hostname or IP address.
- `-M`: module to execute.
{{</details>}}

[^brute-force-wiki]: Contributors to Wikimedia projects. “Brute-Force Search - Wikipedia.” Wikipedia, the Free Encyclopedia, Wikimedia Foundation, Inc., 13 Oct. 2002, https://en.wikipedia.org/wiki/Brute-force_search.
[^cewl]: digininja. “GitHub - Digininja/CeWL: CeWL Is a Custom Word List Generator.” GitHub, https://github.com/digininja/CeWL/.
[^cupp]: Mebus. “GitHub - Mebus/Cupp: Common User Passwords Profiler (CUPP).” GitHub, https://github.com/Mebus/cupp.
[^crunch]: “Crunch - Wordlist Generator Download.” SourceForge, https://sourceforge.net/projects/crunch-wordlist/.
[^john]: magnumripper. “JohnTheRipper.” GitHub, https://github.com/magnumripper/JohnTheRipper.
[^hydra]: Heuse, Marc. “GitHub - Vanhauser-Thc/Thc-Hydra: Hydra.” GitHub, https://github.com/vanhauser-thc/thc-hydra. Accessed 12 May 2020.
[^medusa]: “Foofus Networking Services - Medusa.” Foofus.Net | Foofus.Net Advanced Security Services Forum, http://foofus.net/goons/jmk/medusa/medusa.html.
