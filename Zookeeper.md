Zookeeper
下载地址：
http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz
参考[安装教程](https://www.cnblogs.com/lsdb/p/7297731.html)
### 安装方式
zookeeper有单机、伪集群、集群三种部署方式，可根据自己对可靠性的需求选择合适的部署方式
#### 单机
- 下载安装文件
使用wget工具下载：
```shell
wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz
```
- 解压
```
tar zxf zookeeper-3.4.13.tar.gz -C /aikq/zookeeper
```
- 配置
```
# 进入安装目录
cd /aikq/zookeeper
# 创建数据文件和日志文件目录
mkdir data
mkdir logs
# 进入conf目录下，配置参数文件
cp zoo_sample.cfg zoo.cfg
# 编辑文件zoo.cfg
dataDir=/aikq/zookeeper/zookeeper-3.4.13/data
dataLogDir=/aikq/zookeeper/zookeeper-3.4.13/logs
```
- 启动和停止
```
# 进入bin目录
./zkServer.sh start
./zkServer.sh stop
./zkServer.sh restart
./zkServer.sh status
```

#### 伪集群（同一物理机分不同端口部署）
#### 集群