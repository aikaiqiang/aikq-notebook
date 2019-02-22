背景：之前安装Harbor时默认安装的，服务端口默认占用80和443，后来有同事要用到80端口，为了满足同时的需求，不得不麻烦自己修改端口了。

### 修改docker-compose.yml文件端口映射为1180  —>  80

在Harbor路径下找到文件 docker-compose.yml，我本地路径是 /usr/local/harbor
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/harbor_dir.png)

编辑文件：
> vim docker-compose.yml
```yaml
proxy:
    image: vmware/nginx-photon:1.11.13
    container_name: nginx
    restart: always
    volumes:
      - ./common/config/nginx:/etc/nginx:z
    networks:
      - harbor
    ports:
      - 1180:80
      - 443:443
      - 4443:4443
    depends_on:
      - mysql
      - registry
      - ui
      - log
    logging:
      driver: "syslog"
      options:
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "proxy"
```
### 修改common/templates/registry/config.yml文件仓库地址
>vim common/templates/registry/config.yml
找到如下地方，添加端口号
```yaml
auth:
  token:
    issuer: harbor-token-issuer
    realm: $ui_url:1180/service/token
    rootcertbundle: /etc/registry/root.crt
    service: harbor-registry
```

### 停止harbor，重新启动并生成配置文件
```bash
docker-compose stop
./install.sh
```

### 遇到问题：
1. 停止服务后，在生成配置文件的时候，无法删除容器，提示“磁盘在忙”，由于容器挂载数据卷，无法直接删除。
解决：
- 先查出进程ID 进行杀掉，然后再删除：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/harbor_kill.png)

代码：（example）
```
# grep c413f59ac6f0395634567891119784ad3ec61aad98892624395b57388beb3dae /proc/*/mountinfo
# kill -9 进程ID
```


- 重新运行 install.sh

----
### 修改Harbor域名
```shell
# 进入目录
cd /usr/local/harbor

# 停止Harbor服务
docker-compose stop

# 编辑配置文件
vim harbor.cfg

# 重新初始化
./install.sh 
```

### harbor修改默认默认存储地址：
修改文件 harbor.cfg
```
#The path of secretkey storage
secretkey_path = /自定义数据存储目录 # 默认是 /data
```
修改文件docker-compose.yml（修改原先所有默认为"/data"的volume的挂载路径）
