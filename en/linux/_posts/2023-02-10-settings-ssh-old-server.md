---

layout: post
title: Settings SSH client to connect with old ssh server
author: Luighi Viton-Zorrilla
lang: en
lang-ref: settings-ssh-client
comments: true

---

Sometimes we have some old servers that should be connected to new clients. Then those clients would require some encryption modes that are not available in the client.

```console
$ ssh luighi@10.10.20.31 -p 22
Unable to negotiate with 10.10.20.31 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss
```

### Connecting via ssh client

One way to specify those options are directly in the command line:

```console
$ ssh -o HostKeyAlgorithms=ssh-rsa,ssh-dss -o KexAlgorithms=diffie-hellman-group14-sha1 -p 22 luighi@10.10.20.31
```

### Copying files

To copy files we can use an script that could insert the corresponding options into the rsync tool that is able to synchronize the files.

```bash
#!/bin/bash

LOCAL_FOLDER="/mnt/data/project/"
VM_FOLDER="~/project/"
VM_USER="luighi"
VM_HOST=10.10.20.31
VM_PORT=22

rsync -Pav --delete -e "ssh -o HostKeyAlgorithms=ssh-rsa,ssh-dss -o KexAlgorithms=diffie-hellman-group14-sha1 -p ${VM_PORT}" ${VM_USER}@${VM_HOST}:${VM_FOLDER} ${LOCAL_FOLDER}
```

In the case from the local to the server:

```bash
#!/bin/bash

LOCAL_FOLDER="/mnt/data/project/"
VM_FOLDER="~/project/"
VM_USER="luighi"
VM_HOST=10.10.20.31
VM_PORT=22

rsync -Pav -e "ssh -o HostKeyAlgorithms=ssh-rsa,ssh-dss -o KexAlgorithms=diffie-hellman-group14-sha1 -p ${VM_PORT}" ${LOCAL_FOLDER} ${VM_USER}@${VM_HOST}:${VM_FOLDER}
```

### Using with X2Go

Using with X2go however, it rises errors related to the encryption mode:

![X2Go key error](/assets/images/x2go-key-error.png)

To solve it we use the configuration inside the `~/.ssh/config`

```bash
Host 10.10.20.31
  HostName 10.10.20.31
  User luighi
  HostKeyAlgorithms=ssh-rsa,ssh-dss
  KexAlgorithms=diffie-hellman-group14-sha1
  Port 22
```

After that we can connect normally via X2Go. We can also connect via the command line just specifying the server host to enable connect to it without any option:

```console
$ ssh 10.10.20.31
```
