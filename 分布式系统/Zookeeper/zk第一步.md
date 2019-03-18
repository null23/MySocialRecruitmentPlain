### zk的使用
#### 1.安装
	下载zk并解压
	修改zoo_sample.cfg文件名为zoo.cfg，并修改以下核心配置：

	# 投票选举新leader的初始化时间(以tickTime为单位)
	initLimit=10
	# 数据目录(存储内存数据库快照)
	dataDir=D:/Software/Architecture/zookeeper-3.4.11/data/2
	# 日志目录(存储事务日志)
	dataLogDir=D:/Software/Architecture/zookeeper-3.4.11/logs/3
	# ZooKeeper的最小时间单位(ms)
	tickTime=2000
	# Leader检测Follower可用性心跳的超时时间(以tickTime为单位)
	syncLimit=5
	# 客户端用来连接 ZooKeeper 的端口
	clientPort=2181

#### 2.zk的简单使用
	1.启动bin目录下的zkServer.cmd
	2.为了验证zk启动成功，在启动bin目录下的zkCli.cmd
	3.此时，可以使用ls /命令，查看节点
	4.使用create -e /newNode value创建一个新节点并赋值为value
