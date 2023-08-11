## 升级openssh

### 由于这些中间件日常更新,扫描漏洞会经常出现一些猝不及防的漏洞,所以需要偶尔升级openssh服务,本文都是以离线升级为基础,yum源如果配置好的,可以省去很多麻烦.

#### 下载需要的安装包(此次安装9.3p1)

- [Index of /pub/OpenBSD/OpenSSH/portable/](https://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.3p1.tar.gz)

- 参考文章

  https://www.cnblogs.com/ippondo/p/16573478.html

#### 内网主机升级准备

- 上传文件,解压,新建目录

  ~~~shell
  tar -zxvf /home/user/openssh-9.3p1.tar.gz
  mkdir /usr/local/openssh
  ~~~

- 切换目录

  ~~~shell
  cd openssh-9.3p1/
  
  # 备份
  mv /etc/ssh{,.bak}
  
  # 编译 下载
  ./configure --prefix=/usr/local/openssh --sysconfdir=/etc/ssh --with-openssl-includes=/usr/local/include --with-ssl-dir=/usr/local/lib64 --with-zlib --with-md5-passwords --with-pam && make && make install
  ~~~

- 报错如下

  ~~~shell
  checking whether OpenSSL's headers match the library... no
  configure: error: Your OpenSSL headers do not match your
          library. Check config.log for details.
          If you are sure your installation is consistent, you can disable the check
          by running "./configure --without-openssl-header-check".
          Also see contrib/findssl.sh for help identifying header/library mismatches.
  ~~~

- 删除 多余的openssl，openssl-1.1.1q

- 重新编译

  ~~~shell
  ./configure --prefix=/usr/local/openssh --sysconfdir=/etc/ssh --with-openssl-includes=/usr/local/include --with-ssl-dir=/usr/local/lib64 --with-zlib --with-md5-passwords --with-pam && make && make install
  ~~~

- 编译成功后

  ~~~shell
  echo "UseDNS no" >> /etc/ssh/sshd_config
  echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
  echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config
  echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config
  # 备份原文件
  mv /usr/sbin/sshd{,.bak}
  mv /usr/bin/ssh{,.bak}
  mv /usr/bin/ssh-keygen{,.bak}
  # 创建软连接
  ln -s /usr/local/openssh/bin/ssh /usr/bin/ssh
  ln -s /usr/local/openssh/bin/ssh-keygen /usr/bin/ssh-keygen
  ln -s /usr/local/openssh/sbin/sshd /usr/sbin/sshd
  
  #验证
  ssh -V
  
  # 如果版本没变化，检查版本
  strings /usr/local/sbin/sshd | grep 8.6 
  # 修改版本号
  sed -i 's/OpenSSH_8.6/OpenSSH_9.3/g' /usr/local/sbin/sshd
  # 验证
  sshd -V
  # 如果版本没变化，检查版本
  strings /usr/local/bin/ssh | grep 8.6
  # 修改版本号
  sed -i 's/OpenSSH_8.6/OpenSSH_9.3/g' /usr/local/bin/ssh
  
  # 验证
  ssh -V
  ~~~

### 重新生成启动文件

~~~shell
systemctl disable sshd --now

mv /usr/lib/systemd/system/sshd.service{,.bak}

systemctl daemon-reload
# 没有目录，可不执行
cd /tmp/update

cp -a openssh-9.3p1/contrib/redhat/sshd.init /etc/init.d/sshd

cp -a openssh-9.3p1/contrib/redhat/sshd.pam /etc/pam.d/sshd.pam

chkconfig --add sshd

systemctl enable sshd --now

systemctl start sshd

systemctl status sshd
~~~
