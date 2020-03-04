创建主题：

./kafka-topics.sh --zookeeper localhost:12181 --create --topic lfwer --partitions 10 --replication-factor 2

 

增加主题分区数

./kafka-topics.sh --zookeeper localhost:12181 --alter --topic lfwer --partitions 10

 

删除主题

在server.properties中增加设置，默认未开启

delete.topic.enable=true

重启kafka

./kafka-server-stop.sh

./kafka-server-start.sh -daemon ../config/server.properties

执行删除

./kafka-topics.sh --zookeeper localhost:12181 --delete --topic lfwer

 

生产数据：

./kafka-console-producer.sh  --topic lfwer --broker-list localhost:9092

 

消费数据：

./kafka-console-consumer.sh --topic lfwer --bootstrap-server localhost:9092

 

查看主题在分区中的偏移量 （time为-1时表示最大值，time为-2时表示最小值）

./kafka-run-class.sh kafka.tools.GetOffsetShell --topic lfwer --time -1 --broker-list localhost:9091 --partitions 0

 

查看有多少个组

./kafka-consumer-groups.sh --bootstrap-server localhost:9091 --list

 

查看组消费情况

./kafka-consumer-groups.sh --bootstrap-server localhost:9091 --describe --group my-group