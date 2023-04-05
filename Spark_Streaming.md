# Spark Scala 00X - Spark Streaming

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

Scala, Spark

# How to Complete

Guided steps through creating a stream of data using Spark streaming and structured streaming. Should be completed using the Spark Shell and a linux terminal.

## Spark Streaming

Open a Spark shell and run the following script:



	import org.apache.spark._
	import org.apache.spark.streaming._
	import org.apache.spark.streaming.StreamingContext._ // not necessary since Spark 1.3
	val ssc = new StreamingContext(sc, Seconds(1))
	val lines = ssc.socketTextStream("localhost", 9999)
	val words = lines.flatMap(_.split(" "))
	val pairs = words.map(word => (word, 1))
	val wordCounts = pairs.reduceByKey(_ + _)
	// Print the first ten elements of each RDD generated in this DStream to the console
	wordCounts.print()
	ssc.start()  



In another terminal, open a port to transfer data to this DStream you have ccreated:
*Note: You may have to install netcat on your machine




	nc -lk 9999



Now, when you type data into this terminal, it should appear in the window in which you started your DStream


## Structured Streaming

Open a Spark shell and run the following script:


	import org.apache.spark.sql.functions._
	import org.apache.spark.sql.SparkSession

	val spark = SparkSession.builder.appName("StructuredNetworkWordCount").getOrCreate()
	val lines = spark.readStream.format("socket").option("host", "localhost").option("port", 9999).load()
	val words = lines.as[String].flatMap(_.split(" "))
	val wordCounts = words.groupBy("value").count()
	val query = wordCounts.writeStream.outputMode("complete").format("console").start()
	query.awaitTermination()


In another terminal, open a port to transfer data to this unbounded dataframe you have ccreated:
*Note: You may have to install netcat on your machine




	nc -lk 9999



Now, when you type data into this terminal, it should appear in your spark shell