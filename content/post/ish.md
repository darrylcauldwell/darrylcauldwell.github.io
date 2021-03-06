+++
title = "Running Linux on iPhone"
date = "2020-11-06"
description = "Running Linux on iPhone"
tags = [
    "ios"
]
categories = [
    "handy hacks"
]
thumbnail = "clarity-icons/mobile-144.svg"
+++

iSH is a [project](https://github.com/ish-app/ish) which offers a Linux shell environment on iOS using a usermode x86 emulator.

The emulator is based on Alpine Linux but does not ship with SSH or APK the Alpine Linux package manager. To install the package manager with iSH and ultimately other tools we can download this with Safari from following URL.

```
http://dl-cdn.alpinelinux.org/alpine/latest-stable/main/x86/apk-tools-static-2.10.5-r1.apk
```

The emulator does not by default have access to iCloud Drive files, but this can be mounted by running following.  Note when mounting iCloud Drive you get prompted which folder,  you can select root or Downloads folder where APK install file would be located.

```
mount -t ios . /mnt
```

With the iCloud Drive mounted to emulator we can look to unpack and then create a symbolic link.

```
cd /mnt/Downloads
tar xvzf apk-tools-static-2.10.5-r1.apk -C /
ln -s /sbin/apk.static /sbin/apk
```

With the package manager client installed we can install things like OpenSSH.

```
apk add --no-cache openssh 
```

We can then use the SSH client to connect to any network reachable IP running SSH server.

While exploring iSH I found another interesting thing is that it seems to share the network stack.  So if you run a web server within iSH like:

```
apk add --no-cache python3
echo 'Hello from iSH' >> index.html
python3 -m http.server
```

When you see the ‘Serving HTTP on 0.0.0.0 port 8000’ message, you can then switch to safari and browse localhost.

```
http://127.0.0.1:8000
```

Pretty cool.