### Consul部署安装

#### win环境安装
- 下载
下载地址：https://www.consul.io/downloads.html
- 安装
傻瓜式自定义目录安装即可；
- 设置环境变量
根据上一步安装目录设置：path
- 启动
cmd命令窗口，执行名`consul agent -dev`; dev表示开发模式
cmd 命令窗口执行:
```
consul agent -server ui -bootstrap -client 0.0.0.0 -data-dir="E:\consul" -bind X.X.X.X
```
X.X.X.X为服务器ip,即可使用http://X.X.X.X:8500 访问ui;而不是只能使用localhost连接

#### 集群部署

### 服务管理
官网参考地址：https://www.consul.io/api/agent/service.html
手动删除无效服务
```
http://localhost:8500/v1/agent/service/deregister/服务名称
```
手动删除无效节点
```
http://localhost:8500/v1/agent/force-leave/节点名
```

### Consul + fabio 实现自动服务发现，负载均衡

