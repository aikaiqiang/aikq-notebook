## docker 常用管理命令

### 修改镜像地址

```shell
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://dhq9bx4f.mirror.aliyuncs.com"],
  "insecure-registries":["192.168.0.24:1180"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



### 删除Docker镜像文件image

```shell
# 查看系统镜像 
docker images

# 删除某个镜像 
docker rmi imageID

# 删除全部镜像
docker rmi $(docker images -q)

docker rmi $(docker images | grep magic | awk '{print $3}')
```



### Docker进入正在运行的容器

```shell
# 查看当前系统正在运行的容器
$ sudo docker ps  

# 进入容器
$ sudo docker exec -it 775c7c9ee1e1 /bin/bash
```


### Docker误操作：没有停止容器就删除容器
报错：rpc error: code = 14 desc = grpc: the connection is unavailable
```
# 重启docker
systemctl restart docker
# 以debug模式模式调试容器
docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --shim docker-containerd-shim --runtime docker-runc --debug
```
输入以上命令后就可以使用docker stop 停止容器，再docker rm 删除容器；
这次错误是由于偷懒，直接关闭开启中的容器导致的。以后要先关闭容器，再删除；