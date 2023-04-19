# Spark Scala 00X - Spark Streaming

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

PySpark, Python, Unix

# How to Complete

Guided steps through creating a stream of data using Spark streaming and structured streaming. Should be completed using the Spark Shell and a linux terminal.

## Spark Streaming

Create a Python script with the following code:


	from pyspark import SparkContext
	from pyspark.streaming import StreamingContext

	# Create a local StreamingContext with two working thread and batch interval of 1 second
	sc = SparkContext("local[2]", "NetworkWordCount")
	ssc = StreamingContext(sc, 1)

	# Create a DStream that will connect to hostname:port, like localhost:9999
	lines = ssc.socketTextStream("localhost", 9999)

	# Split each line into words
	words = lines.flatMap(lambda line: line.split(" "))

	# Count each word in each batch
	pairs = words.map(lambda word: (word, 1))
	wordCounts = pairs.reduceByKey(lambda x, y: x + y)

	# Print the first ten elements of each RDD generated in this DStream to the console
	wordCounts.pprint()

	ssc.start()             # Start the computation
	ssc.awaitTermination()  # Wait for the computation to terminate




In another terminal, open a port to transfer data to this DStream you have ccreated:
*Note: You may have to install netcat on your machine




	nc -lk 9999



Now, when you type data into this terminal, it should appear in the window in which you started your DStream


## Structured Streaming

Open a Spark shell and run the following script:


	from pyspark.sql import SparkSession
	from pyspark.sql.functions import explode
	from pyspark.sql.functions import split

	spark = SparkSession \
	    .builder \
	    .appName("StructuredNetworkWordCount") \
	    .getOrCreate()


	# Create DataFrame representing the stream of input lines from connection to localhost:9999
	lines = spark \
	    .readStream \
	    .format("socket") \
	    .option("host", "localhost") \
	    .option("port", 9999) \
	    .load()


	# Split the lines into words
	words = lines.select(
	   explode(
	       split(lines.value, " ")
	   ).alias("word")
	)


	# Generate running word count
	wordCounts = words.groupBy("word").count()


	# Start running the query that prints the running counts to the console
	query = wordCounts \
	    .writeStream \
	    .outputMode("complete") \
	    .format("console") \
	    .start()

	query.awaitTermination()


In another terminal, open a port to transfer data to this unbounded dataframe you have ccreated:
*Note: You may have to install netcat on your machine



	nc -lk 9999



Now, when you type data into this terminal, it should appear in your spark shell