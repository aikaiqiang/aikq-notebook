>本次操作基于CentOS 7.5部署

## 安装Docker

- 利用yum工具安装docker
```bash
yum -y install docker
```

- 启动docker
```bash
systemctl start docker 
```
输入上面命令后，报错如图:

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/1_start_error.png)

根据提示输入命令：
```shell
systemctl status docker.service
```

输出信息如下：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/2_status_docker.png)ng)

输出指出：SELinux 不支持overlay2，所以要关闭SELinux，编辑文件 /etc/sysconfig/docker  ：`vi /etc/sysconfig/docker`

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/3_edit_docker_config.png)

重启docker： `systemctl restart docker`

- 查看docker服务状态
```bash
systemctl status docker
```
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/4_docker_status.png)

到此docker已经安装并成功启动

-----

- 将docker服务加入开启自启动
```bash
systemctl enable docker
```

- 修改docker镜像地址为国内官方地址，重启docker是配置生效；
`vim /etc/docker/daemon.json`

```
{
"registry-mirrors": ["https://registry.docker-cn.com"]
} 
```

- 修改docker镜像和容器的存储位置，Docker默认的镜像和容器存储位置在/var/lib/docker ：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/5_docker_info.png)

编辑文件docker.service，使用-g参数指定存储位置 

```bash
vi /usr/lib/systemd/system/docker.service
```
ExecStart=/usr/bin/dockerd下面添加如下内容 :

--graph  /data/tool/docker  #自定义存储目录

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_image_location.png)

reload配置文件：systemctl daemon-reload

重启docker服务：systemctl restart docker

- 查看linux信息
```bash
#查看linux当前操作系统内核信息
uname -a
#Linux查看当前操作系统版本信息 
cat /proc/version
#Linux查看cpu相关信息，包括型号、主频、内核信息等 
cat /proc/cpuinfo
```


##  安装Rancher

###  docker容器化部署：（[参考连接](https://rancher.com/docs/rancher/v1.6/zh/installing-rancher/installing-server/)）

#### 启动 RANCHER SERVER - 单容器部署 - 使用外部数据库：
```bash
docker run -d --restart=unless-stopped -p 8080:8080 rancher/server \
--db-host 192.168.0.14 --db-port 3306 --db-user root -db-password         Cobbler1234! --db-name cattle
```

cattle 是提前在192.168.0.14上建好的数据库；

- 启动报错：exec: "docker-proxy": executable file not found in $PATH 如图：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/7_docker_run_error.png)

查看下 docker-proxy 的位置： `cat /usr/lib/systemd/system/docker.service | grep prox` 

创建一条软连接到 /usr/bin/ 下： `ln -s /usr/libexec/docker/docker-proxy-current /usr/bin/docker-proxy` 

- 启动报错2：Error response from daemon: shim error: docker-runc not installed on system，如图：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/9_docker_run_error.png)

查看下 docker-proxy 的位置： `cat /usr/lib/systemd/system/docker.service | grep runc` 

创建一条软连接到 /usr/bin/ 下： `ln -s /usr/libexec/docker/docker-runc-current /usr/bin/docker-runc` 

#### 启动 RANCHER SERVER - 单容器部署 (NON-HA)
```bash
docker run -d --restart=unless-stopped -p 8080:8080 rancher/server 
```

- 启动报错：iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 8080 -j DNAT --to-destination 172.17.0.2:8080 ! -i docker0: iptables: No chain/target/match by that name.如图：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/10_docker_run_error.png)

处理报错：
```bash
# 先看能不能ping通网络。若能依次执行以下命令:
pkill docker
iptables -t nat -F
ifconfig docker0 down
brctl delbr docker0
docker -d
# 重启docker服务
systmctl restart docker 
```
重启后，查看docker 容器： `docker ps -a`  如图：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/11_docker_run_success.png)

启动成功，访问http://ip:8080（部署服务器地址:端口）,如图：

![管理界面](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/12_docker_manage_index.png)

### 配置RANCHER

- 访问控制：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/13_manage_controll.png)

目前选择本地身份认证（[参考链接](https://rancher.com/docs/rancher/v1.6/zh/configuration/access-control/#section-3)）：

设置账户/密码：admin/123456

- 添加主机
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/13_manage_add_host.png)


##  部署私有仓库 Docker Registry 

###  安装运行 docker-registry

- ***默认安装***
```bash
docker run -d -p 5000:5000 --restart=always --name registry registry
```
使用官方的 registry 镜像来启动私有仓库。默认情况下，仓库会被创建在***容器***的 /var/lib/registry 目录下;

- ***指定仓库目录安装***
通过 `-v` 参数来将镜像文件存放在本地的指定路径 :

```bash
docker run -d -p 5000:5000 -v /data/tool/docker/registry:/var/lib/registry registry
```

### 查看系统当前的镜像：
```bash
docker images
```
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/14_docker_tag.png)


###  使用 `docker tag` 来标记一个镜像，然后推送它到仓库。例如私有仓库地址为 `127.0.0.1:5000`

格式为 `docker tag IMAGE[:TAG] [REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]`  

```
docker tag a0b9e05b2a03  127.0.0.1:5000/rancher_server:version1.0
```

如上图，多了一个rancher/server 镜像 tag 为 version1.0

上传到私有仓库：`docker push 127.0.0.1:5000/rancher_server:version1.0`

###  用 `curl` 查看仓库中的镜像 ：`curl 127.0.0.1:5000/v2/_catalog`

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/14_docker_registry_1.png)

直接在浏览器访问查看：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/14_docker_registry_2.png)

----

## 【以下是基于本地win10系统安装docker后构建docker镜像】

备注：下文中镜像仓库ip地址修改为 192.168.0.94

###  使用Dockerfile文件构建镜像
- 编写Dockerfile文件
```Dockerfile
FROM java:8
VOLUME /data/container
ADD magic-eureka-center-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
EXPOSE 8761
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

- docker build构建镜像
格式：docker build -t  仓库名称/镜像名称[:标签]   Dockerfile的相对位置

```bash
docker build -t magic/eureka:0.0.1 .
```

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_build.png)

- 查看镜像

```
docker images
```

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_images.png)


### 使用maven插件构建镜像
在pom.xml文件增加插件：
```
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>0.4.13</version>
    <configuration>
        <imageName>${project.artifactId}:${core.version}</imageName>
        <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

在/src/main/docker目录下新建Dockerfile
```Dockerfile
FROM java:8
VOLUME /data/container
ADD magic-eureka-center-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
EXPOSE 8761
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

子模块根目录下 magic-eureka-center/  使用命令构建，运行命令如下：
```bash
mvn clean package docker:build
```

或者 通过IDEA 配置：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/maven_docker_build.png)

在控制台Console可以看到构建成功后的输出信息；

使用 `docker images` 查看新构建的镜像；


### 上传镜像私有Docker Registry
- 为本地镜像打标签
```bash
docker tag magic/eureka:0.0.1 192.168.0.94:5000/magic/eureka:0.0.1
```

- push到仓库
```bash
docker push 192.168.0.94:5000/magic/eureka:0.0.1
```

有点不顺利~~~，报错如下：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_push_error.png)

错误信息：Get https://192.168.0.94:5000/v2/: http: server gave HTTP response to HTTPS client

查询错误原因：[参考博客](https://www.cnblogs.com/hobinly/p/6110624.html)

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_push_solution.png)

根据以上信息，我修改了本地【win10】docker的daemon.json文件 ，再重启服务： 

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_daemon.png))

重新push就可以了，如图：
```bash
docker push 192.168.0.94:5000/magic/eureka:0.0.1
```

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_push_ok.png)


### 从私有docker仓库pull镜像,启动服务
在192.168.0.94机器上从私有docker镜像仓库获取之前上传的镜像，再运行

- pull镜像

```bash
docker pull 192.168.0.94:5000/magic/eureka:0.0.1
```

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_pull_image.png)

地址上加http协议报错：
```bash
docker pull http://192.168.0.94:5000/magic/eureka:0.0.1 
```
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_pull_error.png)

- 运行镜像
```bash
docker run -d -p 8761:8761 192.168.0.94:5000/magic/eureka:0.0.1
```
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_run_ok.png)

查看运行中的容器：`docker ps`
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker_ps.png)




## 使用Docker-Compose 工具编排微服务
###  安装docker-compose  [参考地址](https://docs.docker.com/compose/install/#install-compose)
- 下载最新版本docker-compose
```bash
curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- 添加执行权限
```bash
sudo chmod +x /usr/local/bin/docker-compose
```
安装完成，如图：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker-compose-success.png)


- 安装docker-compose命令补全工具（输入docker-compose后按Tab键提示）

[参考地址](https://docs.docker.com/compose/completion/)

运行如下命令，重新连接终端即可生效
```bash
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.22.0/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```

- 编写docker-compose.yml文件：
目录结构：

![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/docker-compose-mulu.png)

docker-compose.yml 代码:

```yaml
# 表示该docker-compose.yml文件使用的是Version 2 file format
version: '2'
# Version 2 file format 的固定写法，为project定义服务
services:
# 指定服务名称
eureka:
# replace username/repo:tag with your name and image details
#image: username/repo:tag

# build构建镜像，Dockerfile文件相对地址
build: .

# 暴露端口
ports:
- "8761:8761"
expose:
- 8761
```

Dockerfile文件代码：

```Dockerfile
FROM java:8
VOLUME /data/container
ADD target/magic-eureka-center-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
EXPOSE 8761
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

运行构建`docker-compose up` 
后台运行构建`docker-compose up -d` 


### 指定参数启动容器
首先在Dockerfile文件自定义一个参数, 如 APOLLO
Dockerfile文件代码：
```Dockerfile
FROM java:8-alpine
ADD target/magic-eureka-center-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8761
ENV APOLLO=""
#ENTRYPOINT ["java","-Dapollo.meta=http://192.168.0.24:8080","-jar","/app.jar"]
ENTRYPOINT ["sh","-c","java -Dapollo.meta=$APOLLO -Duser.timezone=GMT+08 -jar /app.jar"]
```
启动命令：
```bash
docker run -d -e APOLLO="http://192.168.0.24:8080" 192.168.0.24:1180/magic/magic-eureka-center:new
```