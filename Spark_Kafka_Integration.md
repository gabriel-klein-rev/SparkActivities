# Spark Scala 00X - Kafka Components

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

WSL, Kafka, Spark, Scala

# How to Complete

Guided steps through utilizing spark streaming as a way of ingesting data from a Kafka cluster.


## Start Kafka cluster
// Run the following command to start ZooKeeper:


	bin/zookeeper-server-start.sh config/zookeeper.properties


// There will be a lot of output, and ZooKeeper will be ready in a short time, typically around a second or two.


// Open another terminal session. Change the directory to the kafka directory, and start the Kafka broker:


	cd kafka_2.13-2.6.0
	bin/kafka-server-start.sh config/server.properties


// Open another terminal session and run the kafka-topics command to create a Kafka topic named quickstart-events:


	cd kafka_2.13-2.6.0
	bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092


//Start Producer


	bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mytest

## Create consumer using Spark Streraming

IntelliJ:
Code:
--------


	import org.apache.log4j.{Level, Logger}
	import org.apache.spark.SparkConf
	import org.apache.spark.streaming.kafka.KafkaUtils
	import org.apache.spark.streaming.{Seconds, StreamingContext}
	import org.apache.spark.streaming._
	import org.apache.spark.streaming.kafka
	import org.apache.spark.streaming.StreamingContext._



	object kafkar {

	    def main(args: Array[String]): Unit = {

	    Logger.getLogger("org").setLevel(Level.OFF)
	    Logger.getLogger("akka").setLevel(Level.OFF)

	    println("program started")

	    val conf = new SparkConf().setMaster("local[4]").setAppName("kafkar")
	    val ssc = new StreamingContext(conf, Seconds(2))

	    //my kafka topic name is 'mytest'
	    val kafkaStream = KafkaUtils.createStream(ssc, "localhost:2181","spark-streaming-consumer-group", Map("mytest" -> 5) )
	    kafkaStream.print()
	    ssc.start
	    ssc.awaitTermination()
	  }
	}



Build.sbt:
-------------


	name := "untitled1"

	version := "0.1"

	scalaVersion := "2.11.12"

	libraryDependencies += "org.apache.spark" % "spark-streaming_2.11" % "2.2.0"
	libraryDependencies += "org.apache.spark" % "spark-streaming-kafka-0-8_2.11" % "2.1.0"
	libraryDependencies += "org.apache.spark" %% "spark-core" % "1.6.0"
	libraryDependencies += "org.apache.spark" %% "spark-streaming" % "1.6.0"
	libraryDependencies += "org.apache.spark" %% "spark-streaming-kafka" % "1.6.0"
	libraryDependencies += "org.apache.spark" % "spark-streaming-kafka-0-10_2.11" % "2.2.0"




Ubuntu:
------------


//cd to location where the IntelliJ Spark folder is located where the code and sbt files are created


	cd /mnt/c/Users/jismi/IdeaProjects/untitled1


// compile the sbt file


	sbt compile


//run the sbt file


	sbt run



//Type data in Producer window and the same will be seen in the Spark session
//Result:
	-------------------------------------------
	Time: 1621792088000 ms
	-------------------------------------------
	-------------------------------------------
	Time: 1621792090000 ms
	-------------------------------------------
	(null,d)
	(null,d)
	(null,sd)
	(null,s)
	(null,d)
	(null,sd)


## References:
https://www.confluent.io/blog/set-up-and-run-kafka-on-windows-linux-wsl-2/
https://www.youtube.com/watch?v=2z6scTH_C4c

