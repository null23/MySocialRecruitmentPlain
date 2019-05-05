broker:
topic:
partition:


Windows下Kafka启动，在kafka根目录:
.\bin\windows\kafka-server-start.bat .\config\server.properties

创建topic，进入bin/windows目录：
kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

创建一个producer，进入bin/windows目录：
kafka-console-producer.bat --broker-list localhost:9092 --topic test

创建一个consumer，进入bin/windows目录：
kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning