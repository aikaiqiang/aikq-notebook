RabbitMQ
### RabbitMQ
RabbitMQ 即一个消息队列，主要是用来实现应用程序的异步和解耦，同时也能起到消息缓冲，消息分发的作用；
RabbitMQ是实现AMQP（高级消息队列协议）的消息中间件的一种，最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗；
AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全；
#### 比较重要的概念有 4 个，分别为：虚拟主机，交换机，队列，和绑定
- 虚拟主机：一个虚拟主机持有一组交换机、队列和绑定。为什么需要多个虚拟主机呢？很简单，RabbitMQ当中，用户只能在虚拟主机的粒度进行权限控制。 因此，如果需要禁止A组访问B组的交换机/队列/绑定，必须为A和B分别创建一个虚拟主机。每一个RabbitMQ服务器都有一个默认的虚拟主机“/”
- 交换机：Exchange 用于转发消息，但是它不会做存储 ，如果没有 Queue bind 到 Exchange 的话，它会直接丢弃掉 Producer 发送过来的消息。（这里有一个比较重要的概念：路由键 。消息到交换机的时候，交互机会转发到对应的队列中，那么究竟转发到哪个队列，就要根据该路由键）
- 绑定：也就是交换机需要和队列相绑定，是多对多的关系
#### 交换机
交换机的功能主要是接收消息并且转发到绑定的队列，交换机不存储消息，在启用ack模式后，交换机找不到队列会返回错误。交换机有四种类型：Direct, topic, Headers and Fanout
-  Direct：direct 类型的行为是"先匹配, 再投送". 即在绑定时设定一个 routing_key, 消息的routing_key 匹配时, 才会被交换器投送到绑定的队列中去
-  Topic：按规则转发消息（最灵活）
-  Headers：设置header attribute参数类型的交换机
-  Fanout：转发消息到所有绑定队列
匹配规则：
|类型|描述|
|:---|:---|
|fanout|把所有发送到该Exchange的消息路由到所有与它绑定的Queue中|
|direct|Routing Key==Binding Key（严格匹配）|
|topic|模糊匹配|
|headers|Exchange不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配|


### RabbitMQ下载于安装
官网参考地址：http://www.rabbitmq.com/install-windows-manual.html
rabbitmq是基于erlang开发的，所以安装服务端必须先安装erlang环境
环境：Centos 7 64位
#### 下载安装包
erlang 安装包地址：https://github.com/rabbitmq/erlang-rpm/releases
rabbitmq安装包地址：https://github.com/rabbitmq/rabbitmq-server/releases
下载命令：(先安装好wget命令)
```
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v21.1.4/erlang-21.1.4-1.el7.centos.x86_64.rpm
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.9/rabbitmq-server-3.7.9-1.el7.noarch.rpm
```
#### 安装(yum命令)
yum安装，一直y即可：
```
yum install erlang-21.1.4-1.el7.centos.x86_64.rpm
yum install rabbitmq-server-3.7.9-1.el7.noarch.rpm
```
没有报错表示安装完成，输入命令：
```
netstat -antl | grep 5672
```
显示端口 25672，5672

#### 安装管理界面
```
# 安装
rabbitmq-plugins enable rabbitmq_management
# 查看端口15672存在表示安装成功 
netstat -antl | grep 5672
# 防火墙添加端口
firewall-cmd --zone=public --add-port=5672/tcp --permanent
firewall-cmd --zone=public --add-port=15672/tcp --permanent
```
浏览器输入localhost:15672即可访问，默认用户/密码guest/guest；

#### 开发管理界面远程访问
需要添加配置文件，rabbitmq.config
```
# 新建文件
vim /etc/rabbitmq/rabbitmq.config
# 输入内容,表示对用户admin开启远程访问（此处有问题，测试时全部用户用都能访问）
[
{rabbit,[{loopback_users,["admin"]}]}
].
# 重启服务
systemctl restart rabbitmq-server
或
systemctl stop rabbitmq-server
systemctl start rabbitmq-server
或
service rabbitmq-server stop
service rabbitmq-server start
```

#### 修改添加管理用户
处于安全考虑，默认用户guest都知道密码是guest，所以可以修改guest密码或者新建用户
```
# 修改用户密码
rabbitmqctl change_password guest 密码
# 新建用户admin
rabbitmqctl add_user admin 密码
# 删除用户admin
rabbitmqctl delete_user admin
# 给用户添加tags（Admin/Monitoring/Policymaker/Management/Impersonator/None）
rabbitmqctl set_user_tags admin administrator（）
# 授予角色权限（virtual host）
rabbitmqctl set_permissions -p / admin '.*' '.*' '.*'
# 清除权限目录
rabbitmqctl clear_permissions -p / admin
```

### rabbitmq使用
本次使用结合Spring cloud bus消息总线；

