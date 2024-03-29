## Installation of Kafka in ubuntu 20
## Source https://www.linuxtechi.com/how-to-install-apache-kafka-on-ubuntu/
## https://www.cloudkarafka.com/docs/kafkarestproxy.html
## kafkaUI - docker run -itd -p 9022:9000 -e KAFKA_BROKERCONNECT=192.168.9.140:9092 -e JVM_OPTS="-Xms32M -Xmx64M" -e SERVER_SERVLET_CONTEXTPATH="/" obsidiandynamics/kafdrop
```
apt update
apt install openjdk-11-jdk -y
java -version
wget https://downloads.apache.org/kafka/3.6.1/kafka_2.12-3.6.1.tgz
tar -xvf kafka_2.12-3.6.1.tgz
mv kafka_2.13-3.6.1 /usr/local/kafka


cat /etc/systemd/system/zookeeper.service
[Unit]
Description=Apache Zookeeper server
Documentation=http://zookeeper.apache.org
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
ExecStart=/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties
ExecStop=/usr/local/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target


cat  /etc/systemd/system/kafka.service
[Unit]
Description=Apache Kafka Server
Documentation=http://kafka.apache.org/documentation.html
Requires=zookeeper.service

[Service]
Type=simple
Environment="JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64"
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl enable kafka
systemctl enable zookeeper

cd /usr/local/kafka
$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic sampleTopic
$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092
$ bin/kafka-console-producer.sh --broker-list localhost:9092 --topic sampleTopic
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic sampleTopic --from-beginning
```

```
sudo adduser kafka
sudo adduser kafka sudo
su -l kafka
mkdir ~/Downloads
curl "https://downloads.apache.org/kafka/3.6.1/kafka-3.6.1-src.tgz" -o ~/Downloads/kafka.tgz
mkdir ~/kafka && cd ~/kafka
tar -xvzf ~/Downloads/kafka.tgz --strip 1
vim ~/kafka/config/server.properties
delete.topic.enable = true
log.dirs=/home/kafka/logs


cat /etc/systemd/system/zookeeper.service
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=kafka
ExecStart=/home/kafka/kafka/bin/zookeeper-server-start.sh /home/kafka/kafka/config/zookeeper.properties
ExecStop=/home/kafka/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
##  /etc/systemd/system/kafka.service
```
[Unit]
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=simple
User=kafka
ExecStart=/bin/sh -c '/home/kafka/kafka/bin/kafka-server-start.sh /home/kafka/kafka/config/server.properties > /home/kafka/kafka/kafka.log 2>&1'
ExecStop=/home/kafka/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
```
systemctl start kafka
systemctl start zookeeper

systemctl enable zookeeper
systemctl enable kafka

~/kafka/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic TutorialTopic
```
## REST call to kafka topic
```
curl -X GET http://192.168.9.136:8082/topics/sampleTopic 
```
####################################################################################################
## How to configure Kafka REST Proxy.
```
```
tar -zxvf confluent-7.5.0.tar.gz -C /opt
cat /opt/confluent-7.5.0/etc/kafka-rest/kafka-rest.properties
#
# Copyright 2018 Confluent Inc.
#
# Licensed under the Confluent Community License (the "License"); you may not use
# this file except in compliance with the License.  You may obtain a copy of the
# License at
#
# http://www.confluent.io/confluent-community-license
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OF ANY KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations under the License.
#

#id=kafka-rest-test-server
#schema.registry.url=http://localhost:8081
#zookeeper.connect=localhost:2181
listeners=http://10.60.0.132:8082
bootstrap.servers=PLAINTEXT://localhost:9092
#
# Configure interceptor classes for sending consumer and producer metrics to Confluent Control Center
# Make sure that monitoring-interceptors-<version>.jar is on the Java class path
#consumer.interceptor.classes=io.confluent.monitoring.clients.interceptor.MonitoringConsumerInterceptor
#producer.interceptor.classes=io.confluent.monitoring.clients.interceptor.MonitoringProducerInterceptor
-------------------------------------------------------------------------------------------------------
root@dev-cd-rx-vm:~# cat /etc/systemd/system/kafkarestproxy.service
[Unit]
Description=Kafka Rest Proxy

[Service]
Type=simple
PIDFile=/var/run/kafkarest.pid
User=root
Group=root
ExecStart=/opt/confluent-7.5.0/bin/kafka-rest-start /opt/confluent-7.5.0/etc/kafka-rest/kafka-rest.properties
ExecStop=/opt/confluent-7.5.0/bin/kafka-rest-stop
Restart=on-failure
SyslogIdentifier=kafkarest


[Install]
WantedBy=multi-user.target

-----------------------------------------------------------------
curl -X GET http://localhost:8082/topics/sampleTopic

systemctl enable kafkarestproxy
systemctl start kafkarestproxy
```
## Docker compose
```
version: "2"
services:
  kafdrop:
    image: obsidiandynamics/kafdrop
    restart: "no"
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: "kafka:29092"
    depends_on:
      - "kafka"
  kafka:
    image: obsidiandynamics/kafka
    restart: "no"
    ports:
      - "2181:2181"
      - "9092:9092"
    environment:
      KAFKA_LISTENERS: "INTERNAL://:29092,EXTERNAL://:9092"
      KAFKA_ADVERTISED_LISTENERS: "INTERNAL://kafka:29092,EXTERNAL://localhost:9092"
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: "INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT"
      KAFKA_INTER_BROKER_LISTENER_NAME: "INTERNAL"
      KAFKA_ZOOKEEPER_SESSION_TIMEOUT: "6000"
      KAFKA_RESTART_ATTEMPTS: "10"
      KAFKA_RESTART_DELAY: "5"
      ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: "0"

```
