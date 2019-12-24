# Rancher + K8s
## Docker安装
### 主机无法上网
提示信息：CentOS7用yum安装软件提示 cannot find a valid baseurl for repobase7x86_64
1. 打开 vi /etc/sysconfig/network-scripts/ifcfg-enp4s0（每个机子都可能不一样，但格式会是“ifcfg-e...”）。但内容包含：
```
TYPE=Ethernet #网卡类型 
DEVICE=eth0 #网卡接口名称 
ONBOOT=no #系统启动时是否自动加载 
BOOTPROTO=static #启用地址协议 --static:静态协议 --bootp协议 --dhcp协议 
IPADDR=192.168.1.11 #网卡IP地址 
NETMASK=255.255.255.0 #网卡网络地址 
GATEWAY=192.168.1.1 #网卡网关地址 
HWADDR=00:0C:29:13:5D:74 #网卡设备MAC地址 
BROADCAST=192.168.1.255 #网卡广播地址
```
修改内容：
```
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=dhcp
DNS1=8.8.8.8
DNS2=4.2.2.2
```
或者，打开 vi /etc/resolv.conf新增以下内容
```
nameserver 8.8.8.8
nameserver 4.2.2.2
nameserver 172.19.0.6
nameserver 172.19.0.5
```
2. 重启网络：`service network restart`

### 修改yum源地址为国内地址
执行如下命令：
```
# CentOs yum源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
# Docker-ce源
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
mv docker-ce.repo /etc/yum.repos.d/
# 或者
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum repolist
```
----
分割线
****
### 安装docker
> 获取 docker-ce-18.09.0-3.el7.x86_64.rpm 安装包
```
yum install -y  docker-ce-18.09.0-3.el7.x86_64.rpm 
```
- 更换docker路径
```
# 非首次使用 
cd /var/lib/ && mkdir /home/hdocker && mv /docker/* /home/hdocker && rm -rf docker
# 首次使用
cd /var/lib/ && mkdir /home/hdocker
    ln -s /home/hdocker/ /var/lib/docker 
```
- 系统信息查看
```
# 查看Redhat版本
 cat /etc/redhat-release
# 查看linux当前操作系统内核信息
uname -a 
# Linux查看当前操作系统版本信息
cat /proc/version
# Linux查看cpu相关信息，包括型号、主频、内核信息等
cat /proc/cpuinfo
```
- docker 有问题不想重装：
```
# 停止服务
service docker stop
# 清空docker
rm -rf /var/lib/docker
# 重启服务
service docker start
```

### 安装rancher
- 主机：192.168.0.93
Rancher Server部署:
```
$sudo docker run -d -v /home/rancher:/var/lib/rancher/ --restart=unless-stopped -p 2280:80 -p 443:443 rancher/rancher:stable
```
浏览器输入，host:443访问：
初次访问设置admin用户密码admin_rancher

- 主机：192.168.0.94  (已安装好docker环境)
k8s集群部署：
```
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.1.5 --server https://192.168.0.93 --token zzxfb8g5tzth2gcqpjj2rdm48jrdrk5p56zwssst99d4vl9fdpg2bv --ca-checksum 89d952d259236e2f41a8bf342839e689a08ba23a03e0baa83082218b31af6903 --etcd --controlplane --worker
```