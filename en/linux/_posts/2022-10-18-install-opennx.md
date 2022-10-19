---

layout: post
title: Installing OpenNx in ArchLinux
author: Luighi Viton-Zorrilla
lang: en
lang-ref: install-opennx
comments: true

---

[OpenNx](http://opennx.net/) is a client software to connect to remote desktops as a replacement for
NoMachine's client.

It has precompiled packages ready for different distributions as presented in
its [Download page](http://opennx.net/download.html).

Specifically in ArchLinux there is an entry in its documentation reference which
stands for the NoMachine named as [FreeNX](https://wiki.archlinux.org/title/FreeNX). 
It contains the package for the server/client functions via `nx3-all` or `nomachine`.
However, for the client it suggests using the AUR package `opennx`. It will be
our choice.

## Starting with the AUR package opennx

Normally, packages can be installed by issuing the command (if using a package
manager that search in AUR packages):

```terminal
$ yay -S opennx
```

It will install the [opennx](https://aur.archlinux.org/packages/opennx) 
package from the AUR repositories. 
It has several dependencies, some of them also AUR packages. If the package was
fine, it will be the only step that we will requiere. However, as the last
update in the AUR was in 2017 it has some issues that should be fixed.


## Issues with the original AUR package

The issues that I found trying to install the package are the following:

- **Sources not found in nx-common**. It is a dependency from opennx. It
    downloads two source packages from a mirror, however the original url
    `ftp://ftp.uni-duisburg.de/` not respond.

- **Sources fails to compile**. Once fixed the mirror, there are two moments
    which output errors:

    - Error with the compiling flags

    ```console
g++ -o libXcompsh.so.3.5.0 -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now -Wl,-soname,libXcompsh.so.3 System.o Socket.o Logger.o Runnable.o Process.o Listener.o Connector.o Dispatcher.o Request.o Display.o  -shared
rm -f  libXcompsh.a
ar clq libXcompsh.a System.o Socket.o Logger.o Runnable.o Process.o Listener.o Connector.o Dispatcher.o Request.o Display.o
ar: libdeps specified more than once
make: *** [Makefile:145: libXcompsh.a] Error 1
make: *** Waiting for unfinished jobs....
==> ERROR: A failure occurred in build().
    Aborting...
    ```

    - Error with the `arc4random_stir` reference.

    ```console
gcc -o nxsshd sshd.o auth-rhosts.o auth-passwd.o auth-rsa.o auth-rh-rsa.o sshpty.o sshlogin.o servconf.o serverloop.o auth.o auth1.o auth2.o auth-options.o session.o auth-chall.o auth2-chall.o groupaccess.o auth-skey.o auth-bsdauth.o auth2-hostbased.o auth2-kbdint.o auth2-none.o auth2-passwd.o auth2-pubkey.o monitor_mm.o monitor.o monitor_wrap.o kexdhs.o kexgexs.o auth-krb5.o auth2-gss.o gss-serv.o gss-serv-krb5.o loginrec.o auth-pam.o auth-shadow.o auth-sia.o md5crypt.o audit.o audit-bsm.o platform.o -L. -Lopenbsd-compat/ -Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now -L/usr/lib/openssl-1.0 -lpthread -lssh -lopenbsd-compat     -lcrypto -lz -lnsl  -lcrypt
/usr/bin/ld: sshconnect1.o: in function `ssh_kex':
/home/luighi/.cache/yay/nx-common/src/nxssh/sshconnect1.c:546: undefined reference to `arc4random_stir'
collect2: error: ld returned 1 exit status
make: *** [Makefile:139: nxssh] Error 1
make: *** Waiting for unfinished jobs....
/usr/bin/ld: sshd.o: in function `generate_ephemeral_server_key':
/home/luighi/.cache/yay/nx-common/src/nxssh/sshd.c:411: undefined reference to `arc4random_stir'
/usr/bin/ld: /home/luighi/.cache/yay/nx-common/src/nxssh/sshd.c:411: undefined reference to `arc4random_stir'
/usr/bin/ld: sshd.o: in function `main':
/home/luighi/.cache/yay/nx-common/src/nxssh/sshd.c:1618: undefined reference to `arc4random_stir'
/usr/bin/ld: sshd.o: in function `server_accept_loop':
/home/luighi/.cache/yay/nx-common/src/nxssh/sshd.c:1238: undefined reference to `arc4random_stir'
collect2: error: ld returned 1 exit status
make: *** [Makefile:142: nxsshd] Error 1
==> ERROR: A failure occurred in build().
    Aborting...
    ```

- **Fail compiling opennx**. The `PKGBUILD` has some issues to detect the
    source folder once it is downloaded.

    ```console
==> Starting build()...
/home/luighi/.cache/yay/opennx/PKGBUILD: line 18: cd: too many arguments
==> ERROR: A failure occurred in build().
    Aborting...

    ```

## Fixing issues


### Fixing nx-common package

To fix the issues before we have to first download and try to install the
problematic dependency [nx-common](https://aur.archlinux.org/packages/nx-common).

```console
$ yay -S nx-common
```

Normally, the packages download by `yay` are stored in `~/.cache/yay`. We can
enter there and locate the `nx-common` folder which contains the `PKGBUILD`

Then we can use this patch:

```diff
diff --git i/PKGBUILD w/PKGBUILD
index 132d485..687de52 100644
--- i/PKGBUILD
+++ w/PKGBUILD
@@ -11,8 +11,8 @@ license=('GPL')
 url="http://nomachine.com/"
 depends=('libjpeg-turbo' 'libpng' 'openssl-1.0' 'gcc-libs' 'libxcomp') 
 makedepends=('xorg-server-devel' 'nx-headers')
-source=(ftp://ftp.uni-duisburg.de/X11/NX/sources/$pkgver/nxcompsh-$pkgver-1.tar.gz
-        ftp://ftp.uni-duisburg.de/X11/NX/sources/$pkgver/nxssh-$pkgver-2.tar.gz        
+source=(https://ftp.disconnected-by-peer.at/NX/sources/$pkgver/nxcompsh-$pkgver-1.tar.gz
+        https://ftp.disconnected-by-peer.at/NX/sources/$pkgver/nxssh-$pkgver-2.tar.gz
         nxcompsh-gcc43.patch)
 options=('!libtool')
 md5sums=('84ade443b79ea079380b754aba9d392e'
@@ -24,12 +24,13 @@ build() {
   cd ${srcdir}/nxcompsh
   patch -Np1 -i ${srcdir}/nxcompsh-gcc43.patch
   ./configure --prefix=/usr/lib/nx
+  sed -i "s/ar clq/ar cq/g" Makefile
   make
 
   # nxssh
   cd ${srcdir}/nxssh
   sed -i "s:NX.h:nx/NX.h:g" clientloop.c packet.c proxy.c
-  ./configure --prefix=/usr --sysconfdir=/etc --libexecdir=/usr/lib --with-cppflags=-I/usr/include/openssl-1.0 --with-ldflags='-L/usr/lib/openssl-1.0 -lpthread'
+  ./configure --prefix=/usr --sysconfdir=/etc --libexecdir=/usr/lib --with-cppflags=-I/usr/include/openssl-1.0 --with-ldflags='-L/usr/lib/openssl-1.0 -lbsd -lpthread'
   make
 }

```

Save this as `PKGBUILD.patch` in the same directory as the original `PKGBUILD`.
Then apply changes by:

```console
$ git apply PKGBUILD.patch
```

For more information about how to apply patches you can see the [references
section](#references).

Then you can execute the installing procedure:

```bash
$ makepkg -i
```

### Fixing opennx package

Although it is probably not the best way to address this issue, in this case we
have hard-coded the folder name where the source files are located, 
using this patch:

```diff
diff --git i/PKGBUILD w/PKGBUILD
index cf9d29c..c6e2116 100644
--- i/PKGBUILD
+++ w/PKGBUILD
@@ -15,7 +15,7 @@ source=(http://downloads.sourceforge.net/project/opennx/opennx/CI-source/opennx-
 md5sums=('5271a2430693858803f2e1ca860e5a6c')
 
 build() {
-  cd "$srcdir"/opennx*
+  cd "$srcdir"/opennx-0.16
   ./configure --prefix=/usr \
     --enable-usbip \
     --with-wx-config=wx-config-2.8
@@ -23,7 +23,7 @@ build() {
 }
 
 package() {
-  cd "$srcdir"/opennx*
+  cd "$srcdir"/opennx-0.16
   make DESTDIR="${pkgdir}" install
   make DESTDIR="${pkgdir}" install-man

```

Save as `PKGBUILD.patch` and then apply the patch via:

```console
$ git apply PKGBUILD.patch
```

Finally, install by:

```bash
$ makepkg -i
```

After it, the executable should be ready as:

```console
$ opennx
```

In the specific case of ArchLinux don't forget to exec `rehash` before if you
want to execute the command in the same terminal just after installing it.

## References

- [7 Patch Command Examples to Apply Diff Patch Files in Linux](https://www.thegeekstuff.com/2014/12/patch-command-examples/)
- [How to create and apply a patch with Git Diff and Git Apply](https://www.specbee.com/blogs/how-create-and-apply-patch-git-diff-and-git-apply-commands-your-drupal-website)
