
# Stock Data Engineering Pipeline
This repository demonstrates how to analyze real-time stock market data using Apache Kafka and Amazon Web Serveces. 

## Architecture
![Architecture](https://github.com/Valerie-Fan/stock-data-engineering/assets/164007751/d8051490-f04a-410d-b1c3-0727cddf0018)
## Prerequisites
* Python Environment and basic python knowledge
* An Amazon Web Serveces account
* Basic Knowledge of AWS such as IAM and running EC2 instances<br>
The purpose of this repository is not for learning Python or AWS, but personal note-taking and practicing data pipelines, so I would assume readers of this repo have basic knowledge of those tools.

## Setup
### Setting up EC2 server
* Launch an instance on AWS, move the .pem file to your current workspace,and run the following code to connect it to the local machine.
```bach
#  run this command to ensure your key is not publicly viewable
chmod 400 "<your_instance_name>.pem"
```
```bach
# connect to your instance using its Public DNS
ssh -i "<your_pem_key_.pem>" ec2-user@ec2-<your_public_IPv4_address>.<your_location>.compute.amazonaws.com
```
It should be like this: 
```bach
ssh -i "kafka-stock-market-project.pem" ec2-user@ec2-12-249-9-85.ap-southeast-2.compute.amazonaws.com
```
After connecting to EC2 successfully, install Kafka and Java on the instance by the following command:
```bach 
# downloading Kafka 
wget https://downloads.apache.org/kafka/3.7.0/kafka_2.13-3.7.0.tgz
tar -xvf kafka_2.12-3.3.1.tgz
```
```bach 
# installing Java
sudo yum install java-1.8.0-openjdk
```
```bach 
# check if Java is installed 
java --version
```
If the Java version is not available for the current EC2 instance, you may need to try installing another Java version.
### Launching Zoo-keeper
* Zoo-keeper should be launched before Kafka, since kafka brokers would run based on zoo-keeper instructions.
```bach
# go inside kafka file, change the kafka version if needed, 
cd kafka_2.13-3.7.0

# start zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties
```
* leave the terminal open, open another terminal and check if zookeeper is running
```bash
# run jps
jps
```
### Launching Kafka
* Open a new terminal, before lauching kafka, we need to make sure memory is allocated for running kafka by editting KAFKA_HEAP_OPTS="-Xmx256M -Xms128M":
```bash
nano bin/kafka-server-start.sh
```
```bach
# go inside kafka file, change the kafka version if needed, 
cd kafka_2.13-3.7.0

# and start zookeeper, replace <your_public_IPv4_address> to your intance address
bin/kafka-server-start.sh config/server.properties
```
* Now it is running on private IP, you could un-commend ADVERTISED_LISTENERS so that it could run on puclic IP (this is not recommended and just for this demonstration).
```bash
nano ~/kafka_2.13-3.7.0/config/server.properties
```
* leave the terminal open, open another terminal and check if kafka is running
```bash
# run jps
jps
```
### Creating a topic
* After making sure both zookeeper and kafka are running, we could now open a new terminal and create a topic by the following command:
```bash
# change <your_public_IPv4_address> to create a topic "stock-info"
bin/kafka-topics.sh --create --topic stock-info --bootstrap-server <your_public_IPv4_address>:9092 --replication-factor 1 --partitions 1
```
### Creating a Producer
* Open a new terminal, create a producer by the following commend: 
```bash
# change <your_public_IPv4_address>
bin/kafka-console-producer.sh --topic stock-info --bootstrap-server <your_public_IPv4_address>:9092 
```
### Creating a Consumer
* Open a new terminal, create a consumer by the following commend: 
```bash
# change <your_public_IPv4_address>
bin/kafka-console-consumer.sh --topic stock-info --bootstrap-server <your_public_IPv4_address>:9092
```
* Up for now, we should have 4 terminal running: zookeeper, kafka, producer, and consumer

## Uploading Stock Data to AWS S3
* Practically, you should have some real-time stock index API to load stock information to producer, but they are mostly not free. So here we demontrate the workflow by genarated data stored in indexProcessed.csv
* After running kafka_producer.ipynb and kafka_consumer.ipynb, you should have stock information stored in your S3 bucket.

## Usage
* After loading the data to S3 bucket, you could use Amazon Web Serces such as Glue, Athena and QuickSight to analyze the data.
* You could also do ETL process using DBT and Airflow before uploading it to S3.

## Acknowledgements
* Thank Darshil for his repo and a helpful tutorial on how to launch Kafka on AWS EC2, I have learned a lot and gained much knowledge about real-time data processing from his courses and [![Youtube Channel]([https://img.shields.io/badge/my_portfolio-000?style=for-the-badge&logo=ko-fi&logoColor=white)](https://katherineoelsner.com/](https://www.youtube.com/watch?v=KerNf0NANMo&t=1643s)).
