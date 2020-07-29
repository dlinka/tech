登录

```java
ssh user@host
//主机和服务器用户名相同,可以省略用户名
ssh host
//指定端口号
ssh -p port user@host
```

---

为什么SSH是安全的?

```java
1.远程主机收到用户的登录请求,然后把自己的公钥发给用户
2.用户使用这个公钥,将登录密码加密后,发送回去
3.远程主机用自己的私钥,解密登录密码

//远程主机公钥地址
/etc/ssh/ssh_known_hosts
~/.ssh/known_hosts
```

---

公钥登录

```java
1.用户公钥存储在远程主机上
2.登录的时候,远程主机向用户发送一串随机码
3.用户使用私钥加密随机码,再发送回远程主机
4.远程主机用公钥解密

//生成公钥和私钥
ssh-keygen
//生成的公钥
~/.ssh/id_rsa.pub
//生成的私钥
~/.ssh/id_rsa
//将公钥存储到远程主机
ssh-copy-id user@host
//远程主机上用户的公钥存储地址
~/.ssh/authorized_keys
```

---

