---
title: SMB Protocol Negotiation Failed
bookToc: false
---

# SMB Protocol Negotiation Failed 

Normally SMB takes care of choosing the appropriate protocol for each connection. However, if the offered protocols are out of client's default range, it will return an error message like this: 

```sh
Protocol negotiation failed: NT_STATUS_IO_TIMEOUT
```

## Solution

Edit the connection protocol range in the client configuration file.  
Add `client min protocol` and `client max protocol` settings to `/etc/samba/smb.conf` under `[global]`.

```sh
# /etc/samba/smb.conf
[global]
client min protocol = CORE
client max protocol = SMB3
```
