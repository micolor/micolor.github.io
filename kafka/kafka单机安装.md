# kafka单机安装

## 安装文件上传  （目录/md4g/tools）

- apache-zookeeper-3.6.3-bin.tar.gz
- kafka_2.13-2.8.0.gz

## 解压并修改配置（目录/md4g/kafka）

```shell
#解压并重命名
tar -zxvf /md4g/tools/apache-zookeeper-3.6.3-bin.tar.gz -C /md4g/kafka/
tar -zxvf /md4g/tools/kafka_2.13-2.8.0.gz -C /md4g/kafka/
cd /md4g/kafka/
mv apache-zookeeper-3.6.3-bin/ zookeeper
mv kafka_2.13-2.8.0/ kafka
#复制配置文件
cd zookeeper
mkdir data
cd conf
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
#修改dataDir=/md4g/kafka/zookeeper/data如下图
```

![image-20211027111006110](images/kafka%E5%8D%95%E6%9C%BA%E5%AE%89%E8%A3%85/image-20211027111006110.png)

## 启动zookeeper，并查看是否安装成功

```shell
/md4g/kafka/zookeeper/bin/zkServer.sh start
/md4g/kafka/zookeeper/bin/zkCli.sh
```

![image-20211027111846747](images/kafka%E5%8D%95%E6%9C%BA%E5%AE%89%E8%A3%85/image-20211027111846747.png)

## kafka配置

```shell
cd /md4g/kafka/kafka/
mkdir data
vim server.properties
#修改：log.dirs=/md4g/kafka/kafka/data 如下图
```

![image-20211027112243156](images/kafka%E5%8D%95%E6%9C%BA%E5%AE%89%E8%A3%85/image-20211027112243156.png)

## 启动kafka

```shell
nohup /md4g/kafka/kafka/bin/kafka-server-start.sh /md4g/kafka/kafka/config/server.properties &
```

## 创建topic测试

```shell
cd /md4g/kafka/kafka/bin
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

## kafka tools 连接

### 安装kafka-tools工具新建连接

File-》Add new connection...

![image-20211027113216300](images/kafka%E5%8D%95%E6%9C%BA%E5%AE%89%E8%A3%85/image-20211027113216300.png)

![image-20211027113243372](images/kafka%E5%8D%95%E6%9C%BA%E5%AE%89%E8%A3%85/image-20211027113243372.png)