# kafka 安装配置使用

## zookeeper 安装
```
wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.5.6/apache-zookeeper-3.5.6-bin.tar.gz && tar zxvf apache-zookeeper-3.5.6-bin.tar.gz && cd apache-zookeeper-3.5.6-bin && cd conf
cp zoo_sample.cfg zoo.cfg && vim zoo.cfg

启动zookeeper
```
./bin/zkServer.sh start
```
zookeeper 默认AdminServer 占用8080端口，更改端口
```
admin.serverPort=8888
```

## 安装kafka
```
1.下载kafka，解压
```
wget https://www.apache.org/dyn/closer.cgi?path=/kafka/2.4.0/kafka_2.13-2.4.0.tgz -O kafka_2.13-2.4.0.tgz && tar zxvf kafka_2.13-2.4.0.tgz
```
3.进入kafka的conf目录，修改相关配置文件
```
修改server.properties
```
4.启动kafka
```
./bin/kafka-server-start.sh -daemon ./config/server.properties
```
4.先启动zooke
```
zkServer.sh start
```
6.创建topic
```
./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
7.查看topic列表
```
./bin/kafka-topics.sh --list --zookeeper localhost:2181
```
8.创建生产者
```
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```
9.创建消费者
```
#0.9版本之前：
./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
#0.9版本之后：
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```


## 配置
