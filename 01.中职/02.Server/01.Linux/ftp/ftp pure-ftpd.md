# 问题：
- 1.配置linux1为ftp服务器，新建本地用户xiaoming,本地用户登录ftp后的目录/var/ftp/pub,可以上传下载。
- 2.配置ftp虚拟用户认证模式，虚拟用户ftp1和ftp2映射为ftp，ftp1登录ftp后的目录为/var/ftp/vdir/ftp1，可以上传下载，ftp2登录ftp的目录为/var/ftp/vdir/ftp2，仅有下载权限。

### 解：
- 先传一个安装包
```
scp -p pure-ftpd-1.0.52.tar.gz root@10.10.120.101:/opt
```

```
yum install tar* gcc* ftp* -y
cd /opt
tar -zxvf pure-ftpd-1.0.52.tar.gz
cd pure-ftpd-1.0.52
./configure --with-puredb
make
make install
mkdir -p /var/ftp/vdir/ftp1
mkdir -p /var/ftp/vdir/ftp2
chown ftp:ftp /var/ftp/vdir
chown ftp:ftp /var/ftp/vdir/*
chmod 555 /var/ftp/vdir/ftp2
pure-pw useradd ftp1 -u ftp -g ftp -d /var/ftp/vdir/ftp1  (给ftp1用户密码)
pure-pw useradd ftp2 -u ftp -g ftp -d /var/ftp/vdir/ftp2  (给ftp2用户密码)
useradd xiaoming
passwd xiaoming
pure-pw mkdb /etc/pureftpd.pdb
vi /opt/pure-ftpd-1.0.52/pure-ftpd.conf

77 NoAnonymous yes
125 PureDB /etc/pureftpd.pdb
241 MinUID 10

pure-ftpd pure-ftpd.conf
```


### 问题：
- 1.配置ftp虚拟用户认证模式，虚拟用户ftp1和ftp2映射为ftpuser，ftp1登录ftp后的目录为/home/ftpuser/ftp1，可以上传下载，ftp2登录ftp的目录为/home/ftpuser/ftp2，仅有下载权限。

### 解：
- 先传一个安装包
```
scp -p pure-ftpd-1.0.52.tar.gz root@10.10.120.101:/opt
```

```
yum install tar* gcc* ftp* -y
cd /opt
tar -zxvf pure-ftpd-1.0.52.tar.gz
cd pure-ftpd-1.0.52
./configure --with-puredb
make
make install
useradd ftpuser
mkdir -p /home/ftpuser/ftp1
mkdir -p /home/ftpuser/ftp2
chown ftp:ftp /home/ftpuser/*
chmod 555 /home/ftpuser/ftp2
pure-pw useradd ftp1 -u ftp -g ftp -d /home/ftpuser/ftp1  (给ftp1用户密码)
pure-pw useradd ftp2 -u ftp -g ftp -d /home/ftpuser/ftp2  (给ftp2用户密码)
pure-pw mkdb /etc/pureftpd.pdb
vi /opt/pure-ftpd-1.0.52/pure-ftpd.conf

77 NoAnonymous yes
125 PureDB /etc/pureftpd.pdb

pure-ftpd pure-ftpd.conf
