---
title: "SSH"
date: 2018-04-24 14:42
tag: SSH
category: Others
---

[TOC]

# SSH

## SSH Public Key

> 用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。

### Process
```shell
ssh-keygen
# Generate Public Key

ssh-copy-id user@host
# publish the Public Key to the remote host
```
Confirm that:
```shell
# In /etc/ssh/sshd_config of Remote Host
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```
Restart ssh service of Remote Host
```shell
service ssh restart # ubuntu
/etc/init.d/ssh restart # Debian
```
### `authorized_keys`
Remote Host saves the public keys into `$HOME/.ssh/authorized_keys`.
Public Keys are actually a piece of **String**, which could be appended at the end of `authorized_keys`.

`ssh-copy-id` equals to
```shell
ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
```
## Remove Actions

### File Transfer

```shell
cd && tar czv src | ssh user@root 'tar xz'
# Copy $HOME/src to $HOME/src (Remote)

ssh user@host 'tar cz src' | tar xzv
# Copy $HOME/rc to .

ssh user@host 'ps ax | grep [h]ttpd'
```

### Port Forwarding

#### Local Port Binding

```shell
ssh -D 8080 user@host
```
#### Local Forwarding

```shell
ssh -L <Local Port>:<Target Host>:<Target Port> AgentHost
# Target Host: regarding to Agent Host
```

```shell
ssh -L 5900:localhost:5900 host3
# Local:5900 to Host3:5900 (local host is to host3)
```

```shell
ssh -L 9001:host2:22 host3
ssh -p 9001 localhost # Aquivilant to ssh host2
```

#### Other Parameters

- `-D` 只打开远程主机，不打开远程Shell
- `-T` 不为当前连接分配TTY
- `-f` ssh连接成功后转入后台运行

## Resources

- [阮一峰ssh笔记](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
