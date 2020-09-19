---
title: SSH
---

SSH （Secure Shell，安全外壳协议），用于计算机之间的加密登录。

传统的网络服务程序，如：ftp、pop 和 telnet 在本质上都是不安全的，因为它们在网络上用明文传送口令和数据，别有用心的人非常容易就可以截
获这些口令和数据。

## 中间人攻击

SSH之所以能够保证安全，原因在于它采用了公钥加密。

整个过程是这样的：

1. 远程主机收到用户的登录请求，把自己的公钥发给用户。
2. 用户使用这个公钥，将登录密码加密后，发送回来。
3. 远程主机用自己的私钥，解密登录密码，如果密码正确，就同意用户登录。

这个过程本身是安全的，但是实施的时候存在一个风险：如果有人截获了登录请求，然后冒充远程主机，将伪造的公钥发给用户，那么用户很难辨别真伪。
因为不像 https 协议，SSH 协议的公钥是没有证书中心（CA）公证的，也就是说，都是自己签发的。

可以设想，如果攻击者插在用户与远程主机之间（比如在公共的 wifi 区域），用伪造的公钥，获取用户的登录密码。再用这个密码登录远程主机。
这种风险就是的"中间人攻击"（Man-in-the-middle attack）。

## 口令登录

如果你是第一次登录远程主机，系统会出现下面的提示：

```sh
$ ssh user@host
The authenticity of host '16.187.189.94 (16.187.189.94)' can't be established.
ECDSA key fingerprint is MD5:b0:4f:bb:ef:80:aa:07:5f:08:f2:81:5f:5f:9d:73:4f.
Are you sure you want to continue connecting (yes/no)?
```

这段话的意思是，无法确认 host 主机的真实性，只知道它的公钥指纹，是否继续？
**公钥指纹**是指公钥长度较长（这里采用 RSA 算法，长达 1024 位），很难比对，所以对其进行哈希计算，将它变成一个 128 位的指纹。例如
 `b0:4f:bb:ef:80:aa:07:5f:08:f2:81:5f:5f:9d:73:4f`，再进行比较，就容易多了。

用户怎么知道远程主机的公钥指纹应该是多少？并没有好办法，远程主机必须在自己的网站上贴出公钥指纹，以便用户自行核对。

如果用户决定接受这个远程主机的公钥，输入 yes。系统会出现一句提示，然后，会要求输入密码。：

```sh
Warning: Permanently added '16.187.189.94' (RSA) to the list of known hosts.
Password: (enter password)
```

密码正确，就可以登录了。

## 公钥登录

使用密码登录，每次都必须输入密码，非常麻烦。可以通过公钥登录。

**公钥登录**：

1. 用户将自己的公钥储存在远程主机上。
2. 登录的时候，远程主机会向用户发送一段随机字符串。
3. 用户收到随机字符串，用自己的私钥加密后，再发给远程主机。
4. 远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录 shell，不再要求密码。

用户可以使用 `ssh-keygen` 密钥对，生成时可以对私钥设置口令（passphrase）。一般密钥对会放在 `$HOME/.ssh/` 目录下，会新生成两个
文件：`id_rsa.pub` 和 `id_rsa`。前者是公钥，后者是私钥。

将公钥传送到远程主机上面：

1. 查看是否存在 `$HOME/.ssh/authorized_keys` 文件，不存在则创建该文件：
2. 将生成的 `id_rsa.pub` 文件内容粘贴到 `authorized_keys` 文件。

然后再登录，就不需要输入密码了。

如果还是不行，就打开远程主机的 `/etc/ssh/sshd_config` 这个文件，去掉如下三行注释#：

```sh
#RSAAuthentication yes
#PubkeyAuthentication yes
#AuthorizedKeysFile     .ssh/authorized_keys
```

重启 sshd 服务：

```sh
service sshd restart

# CentOS
systemctl restart sshd
```
