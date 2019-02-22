### gitbook使用教程
GitBook 是基于 Node.js，所以我们首先需要安装 Node.js ;[下载地址](https://nodejs.org/en/download/)，找到对应平台的版本安装即可

安装node会自动安装好npm（node package manage）
全局安装gitbook构建工具
```
npm install -g gitbook-cli
```
#### 构建第一本book
新建文件夹book，进入book目录下，打开命令编辑器，使用如下命令构建：
```
gitbook init
```
book目录下多了2个文件：README.md 和 SUMMARY.md
#### 编辑目录文件SUMMARY.md
```
* [前言](README.md)
* [第一章 1](目标文件路径)
  * [第一节 1.1]()
  * [第二节 1.2]()
* [第二章 2]()
  * [第一节 2.1]()
  * [第一节 2.2]()
  * [第一节 2.3]()

```

#### 本地预览
启动本地服务：
```
# 默认端口（4000）启动
gitbook serve
or
# 指定端口启动
gitbook serve port 1000
```
book目录下会生成一个_book文件夹（html静态页面）
浏览器输入localhost:4000预览