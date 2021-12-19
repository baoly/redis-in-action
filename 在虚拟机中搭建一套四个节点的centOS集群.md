# 配置Virtual Box

## 基本配置

- 设置内存和硬盘

- 配置虚拟机网络 连接方式选择 桥接网卡

- 配置虚拟机网路

- 编辑网卡

  ```bash
  vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
  未进入编辑模式，按两次D删除该行
  删除 UUID
  删除 NM-CONTROLED
  
  文件内容
  DEVICE=enp0s3
  TYPE=Ethernet
  BOOTPROTO=dhcp
  ONBOOT=yes
  重启网卡(这里的目的是让它动态的去给虚拟机分配一个ip地址)
  service network restart
  测试 
  ping baidu.com
  安装 ifconfig(centOS7原生没有这个命令)
  sudo yum install -y net-tools
  192.168.*.*
  此时宿主机和虚拟机可以互相ping通
  编辑 vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
  固化ip
  IPADDR=192.168.*.*
  NETMASK=255.255.255.0(固定)
  GATEWAY=192.168.*.** (来自宿主机，宿主机的网关是多少，这里就写多少)
  BOOTPROTO=static
  重启网卡
  service network restart
  配置host文件
  vi /etc/hosts
  固化ip 和虚拟机名字做映射
  192.168.*.* eshop-cache01
  关闭宿主机防火墙
  关闭虚拟机防火墙(我是centOS7)
  systemctl stop firewalld.service            #停止firewall
  systemctl disable firewalld.service        #禁止firewall开机启动
  
  #清除安装的包文件
  yum clean all
  #测试yum是否起作用了
  yum makecache  #如果这里执行失败，看看虚拟机是不是能ping通baidu的域名
  #安装wget
  yum install -y wget
  #删除 /usr/local下的东西
  #下载jdk-8u301-linux-x64.rpm 传到虚拟机
  #安装jdk
  rpm -ivh jdk-8u301-linux-x64.rpm
  #删除 jdk安装包
  rm -rf dk-8u301-linux-x64.rpm
  #配置环境变量
  vi ~/.bashrc
  # .bashrc
  export JAVA_HOME=/usr/local/latest
  export PATH=$PATH:$JAVA_HOME/bin
  #保存退出后 执行
  source ~/.bashrc
  #验证是否安装成功
  java -version
  # 查看linux 版本
  cat /etc/redhat-release
  #解压perl
  tar -zxvf perl-5.16.1.tar.gz
  #删除压缩包
  rm -rf perl-5.16.1.tar.gz
  #安装gcc
  yum install -y gcc
  #进入解压perl的目录
  cd perl-5.16.1
  #执行命令 注意大小写，然后等待
  ./Configure -des -Dprefix=/usr/local/perl
  #执行命令
  make && make test && make install
  #查看 perl版本
  perl -v
  ```

- 最后按照以上步骤，再配置三台一模一样的虚拟机

- 在每台机器hosts文件里，配置好所有的机器的ip和hosts映射关系

- 每台机器 配置ssh免密互相通信

  ```bash
  ssh-keygen -t rsa
  cd /root/.ssh
  cp id_rsa.pub authorized_keys
  #测试
  ssh eshop-cache01 输入yes
  #接下来 ssh连接就不需要密码了
  ssh eshop-cache01 连接成功
  #退出 ssh连接
  logout
  #配置和其它机器的免密登录
  #例如 ssh-copy-id -i eshop-cache02, 将eshop-cache01 的公匙拷贝到eshop-cache02的authorized_keys 中
  # 在234上执行,把自己的id_rsa.pub 拷贝到1的authorized_keys中 
  ssh-copy-id -i eshop-cache01
  #上面的是把1的公匙分别拷贝到2，3，4接下来，把2,3,4的公匙分别拷贝到1 此时1有四个公匙 1234 再把一的id_rsa.pub 拷贝到其它三台机器上
  scp authorized_keys eshop-cache02:/root/.ssh
  scp authorized_keys eshop-cache03:/root/.ssh
  scp authorized_keys eshop-cache04:/root/.ssh
  ```

  

## 问题处理

##### 关于无法创建虚拟电脑的解决办法

- 下载最新的virtual box

##### install centos 无反应

- 新建虚拟机时选32位操作系统，但是存储介质选择的是64位的存储介质，会出现这个问题，解决办法是，新建虚拟机的位数要和存储介质一致或者向下兼容

##### ifconfig command not found

- ``` bash
  cd /sbin
  ls | grep 'ifconfig'
  如果没有,则
  sudo yum install -y net-tools
  ```

##### 虚拟机可以ping通ip但是ping不通域名的解决办法

```bash
vi /etc/resolv.conf
输入
nameserver 114.114.114.114
```

##### make && make test && make install 

make test 命令报错，就单独执行make  install

##### 为什么要装perl

java+nginx+lua 需要perl