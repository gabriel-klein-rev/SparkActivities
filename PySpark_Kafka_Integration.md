# PySpark 00X - Kafka with Spark Streaming

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

WSL, Kafka, Spark, Python

# How to Complete

Guided steps through utilizing spark streaming as a way of ingesting data from a Kafka cluster.


## Start Kafka cluster
Run the following command to start ZooKeeper:


	bin/zookeeper-server-start.sh config/zookeeper.properties


There will be a lot of output, and ZooKeeper will be ready in a short time, typically around a second or two.


Open another terminal session. Change the directory to the kafka directory, and start the Kafka broker:


	cd kafka_2.13-2.6.0
	bin/kafka-server-start.sh config/server.properties


Open another terminal session and run the kafka-topics command to create a Kafka topic named quickstart-events:


	cd kafka_2.13-2.6.0
	bin/kafka-topics.sh --create --topic topic1 --bootstrap-server localhost:9092


Start Producer


	bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mytest

## Create consumer using Spark Streraming

Open up a pyspark-shell instance. Run one of the following scripts to suit your needs:

	# Subscribe to 1 topic
	df = spark \
	  .readStream \
	  .format("kafka") \
	  .option("kafka.bootstrap.servers", "localhost:9092") \
	  .option("subscribe", "topic1") \
	  .load()
	df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

_______________________________

	# Subscribe to 1 topic, with headers
	df = spark \
	  .readStream \
	  .format("kafka") \
	  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
	  .option("subscribe", "topic1") \
	  .option("includeHeaders", "true") \
	  .load()
	df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)", "headers")


_______________________________

	# Subscribe to multiple topics
	df = spark \
	  .readStream \
	  .format("kafka") \
	  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
	  .option("subscribe", "topic1,topic2") \
	  .load()
	df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")


_______________________________

	# Subscribe to a pattern
	df = spark \
	  .readStream \
	  .format("kafka") \
	  .option("kafka.bootstrap.servers", "host1:port1,host2:port2") \
	  .option("subscribePattern", "topic.*") \
	  .load()
	df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")

_______________________________

Now, you should be able to consume data typed into the kafka producer window and see it in the output of your spark-shell script.

## References

https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html