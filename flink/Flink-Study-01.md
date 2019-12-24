Flink
[官网](https://flink.apache.org/)
[翻译文档1.6](http://flink-cn.shinonomelab.com/)
[翻译文档1.7](https://flink.sojb.cn/)
基于条件：已经安装好JDK环境
## 安装
### windows环境
- 下载资源文件，[地址](https://flink.apache.org/downloads.html)
下载对应的安装包，解压即可使用，进入bin目录下，点击bat后缀文件即可启动

- 设置环境变量
FLINK_HOME：flink解压目录  #配置Path，增加 %FLINK_HOME%\bin
FLINK_CONF_DIR: flink配置文件夹conf目录

- 查看版本：`flink --version`

- 启动flink
进入bin目录下，点击运行 start-cluster.bat；弹出日志输出窗口
浏览器输入：local host:8081 如图：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/20190222171343.png)


### linux环境
#### linux 安装 ncat / nc
[参考连接](https://linux.cn/article-9190-1.html)
·yum install nmap-ncat -y·


## 编写本地流处理demo
### 新建flink-demo项目
IDEA new project 选择maven项目，如图：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/20190222171900.png)

### 编写测试类
新建类文件SocketTextStreamWordCount.java
```Java
package com.aikq.flink;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 *  E
 * @author aikq
 * @date 2019年02月22日 14:47
 */
public class SocketTextStreamWordCount {

	public static void main(String[] args) throws Exception {
		// 参数校验
		if (args.length != 2){
			System.out.println("USAGE:\nSocketTextStreamWordCount <hostname> <port>");
			return;
		}

		String hostname = args[0];
		Integer port = Integer.parseInt(args[1]);

		final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

		//获取数据
		DataStreamSource<String> stream = env.socketTextStream(hostname, port);

		//计数
		SingleOutputStreamOperator<Tuple2<String, Integer>> sum = stream.flatMap(new LineSplitter())
				.keyBy(0)
				.sum(1);

		sum.print();

		env.execute("Java WordCount from SocketTextStream Demo");
	}

	public static final class LineSplitter implements 				FlatMapFunction<String, Tuple2<String, Integer>> {
		@Override
		public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) {
			String[] tokens = s.toLowerCase().split("\\W+");
			for (String token: tokens) {
				if (token.length() > 0) {
					collector.collect(new Tuple2<String, Integer>(token, 1));
				}
			}
		}
	}
}

```
### 打包编译
在项目根目录下，使用命令打包：`mvn clean package -Dmaven.test.skip=true`

或者利用IDE，如图：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/20190222172615.png)
1：忽略单元测试
package：执行打包命令，生成jar包，如图：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/20190222175501.png)


### 开启本地监听端口
windows需要安装netcat，[下载地址](https://eternallybored.org/misc/netcat/)
- 下载安装包
- 解压
- nc命令
Windows OS环境下使用nc命令，实现TCP方式监听服务器端9000端口
服务器端命令：`nc.exe -l -p [端口号]`
客户端命令：`nc.exe [服务器端IP地址] [端口号]`
Windows OS环境下使用nc命令，实现UDP方式监听服务器端9000端口
服务器端命令：`nc.exe -lu -p [端口号]`
客户端命令：`nc.exe -u [服务器端IP地址] [端口号]`
如下图：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/20190222174153.png)


## flink运行flink-demo.jar
打开cmd，执行命令
```bash
flink run -c com.aikq.flink.SocketTextStreamWordCount flink-demo-1.0-SNAPSHOT.jar 127.0.0.1 9000
```
`com.aikq.flink.SocketTextStreamWordCount`必须是对应类含包名的全路径
运行成功后可以在flink管理界面看到新增一个job，如图：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/20190222175135.png)

然后再nc监听的9000端口窗口输入测试字符串，在日志窗口有对应输出，如图：
![](https://raw.githubusercontent.com/aikaiqiang/aikq-blog-comments/master/notepic/20190222175032.png)