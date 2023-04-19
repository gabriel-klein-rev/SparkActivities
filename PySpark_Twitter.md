# PySpark 00X - Twitter Application

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

Python, PySpark, Twitter Dev Credentials

# How to Complete

Follow the steps below to complete the Twitter Application


# Register Twitter App

In order to get tweets from Twitter, you need to register on [TwitterApps](https://apps.twitter.com) by clicking on “Create new app” and then fill the below form click on “Create your Twitter app.”

Second, go to your newly created app and open the “Keys and Access Tokens” tab. Then click on “Generate my access token.”

# Building Twitter http client

First, let’s create a file called twitter_app.py and then we’ll add the code as below.

	import socket
	import sys
	import requests
	import requests_oauthlib
	import json

And add the variables that will be used in OAuth for connecting to Twitter as below:

	# Replace the values below with yours
	ACCESS_TOKEN = 'YOUR_ACCESS_TOKEN'
	ACCESS_SECRET = 'YOUR_ACCESS_SECRET'
	CONSUMER_KEY = 'YOUR_CONSUMER_KEY'
	CONSUMER_SECRET = 'YOUR_CONSUMER_SECRET'
	my_auth = requests_oauthlib.OAuth1(CONSUMER_KEY, CONSUMER_SECRET,ACCESS_TOKEN, ACCESS_SECRET)

Now, we will create a new function called get_tweets that will call the Twitter API URL and return the response for a stream of tweets.

	def get_tweets():
		url = 'https://stream.twitter.com/1.1/statuses/filter.json'
		query_data = [('language', 'en'), ('locations', '-130,-20,100,50'),('track','#')]
		query_url = url + '?' + '&'.join([str(t[0]) + '=' + str(t[1]) for t in query_data])
		response = requests.get(query_url, auth=my_auth, stream=True)
		print(query_url, response)
		return response

Then, create a function that takes the response from the above one and extracts the tweets’ text from the whole tweets’ JSON object. After that, it sends every tweet to Spark Streaming instance (will be discussed later) through a TCP connection.

	def send_tweets_to_spark(http_resp, tcp_connection):
		for line in http_resp.iter_lines():
	    	try:
	        	full_tweet = json.loads(line)
	        	tweet_text = full_tweet['text']
	        	print("Tweet Text: " + tweet_text)
	        	print ("------------------------------------------")
	        	tcp_connection.send(tweet_text + '\n')
	    	except:
	        	e = sys.exc_info()[0]
	        	print("Error: %s" % e)

Now, we’ll make the main part which will make the app host socket connections that spark will connect with. We’ll configure the IP here to be localhost as all will run on the same machine and the port 9009. Then we’ll call the get_tweets method, which we made above, for getting the tweets from Twitter and pass its response along with the socket connection to send_tweets_to_spark for sending the tweets to Spark.

	TCP_IP = "localhost"
	TCP_PORT = 9009
	conn = None
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.bind((TCP_IP, TCP_PORT))
	s.listen(1)
	print("Waiting for TCP connection...")
	conn, addr = s.accept()
	print("Connected... Starting getting tweets.")
	resp = get_tweets()
	send_tweets_to_spark(resp, conn)

# Setting Up Our Apache Spark Streaming Application

Let’s build up our Spark streaming app that will do real-time processing for the incoming tweets, extract the hashtags from them, and calculate how many hashtags have been mentioned. Create a new python file:

	from pyspark import SparkConf,SparkContext
	from pyspark.streaming import StreamingContext
	from pyspark.sql import Row,SQLContext
	import sys
	import requests
	# create spark configuration
	conf = SparkConf()
	conf.setAppName("TwitterStreamApp")
	# create spark context with the above configuration
	sc = SparkContext(conf=conf)
	sc.setLogLevel("ERROR")
	# create the Streaming Context from the above spark context with interval size 2 seconds
	ssc = StreamingContext(sc, 2)
	# setting a checkpoint to allow RDD recovery
	ssc.checkpoint("checkpoint_TwitterApp")
	# read data from port 9009
	dataStream = ssc.socketTextStream("localhost",9009)


Now for the transform logic:

	# split each tweet into words
	words = dataStream.flatMap(lambda line: line.split(" "))
	# filter the words to get only hashtags, then map each hashtag to be a pair of (hashtag,1)
	hashtags = words.filter(lambda w: '#' in w).map(lambda x: (x, 1))
	# adding the count of each hashtag to its last count
	tags_totals = hashtags.updateStateByKey(aggregate_tags_count)
	# do processing for each RDD generated in each interval
	tags_totals.foreachRDD(process_rdd)
	# start the streaming computation
	ssc.start()
	# wait for the streaming to finish
	ssc.awaitTermination()

The updateStateByKey takes a function as a parameter called the update function. It runs on each item in RDD and does the desired logic.

In our case, we’ve created an update function called aggregate_tags_count that will sum all the new_values for each hashtag and add them to the total_sum that is the sum across all the batches and save the data into tags_totals RDD.

	def aggregate_tags_count(new_values, total_sum):
		return sum(new_values) + (total_sum or 0)

Then we do processing on tags_totals RDD in every batch in order to convert it to temp table using Spark SQL Context and then perform a select statement in order to retrieve the top ten hashtags with their counts and put them into hashtag_counts_df data frame.

	def get_sql_context_instance(spark_context):
		if ('sqlContextSingletonInstance' not in globals()):
	        globals()['sqlContextSingletonInstance'] = SQLContext(spark_context)
		return globals()['sqlContextSingletonInstance']
	def process_rdd(time, rdd):
		print("----------- %s -----------" % str(time))
		try:
	    	# Get spark sql singleton context from the current context
	    	sql_context = get_sql_context_instance(rdd.context)
	    	# convert the RDD to Row RDD
	    	row_rdd = rdd.map(lambda w: Row(hashtag=w[0], hashtag_count=w[1]))
	    	# create a DF from the Row RDD
	    	hashtags_df = sql_context.createDataFrame(row_rdd)
	    	# Register the dataframe as table
	    	hashtags_df.registerTempTable("hashtags")
	    	# get the top 10 hashtags from the table using SQL and print them
	    	hashtag_counts_df = sql_context.sql("select hashtag, hashtag_count from hashtags order by hashtag_count desc limit 10")
	    	hashtag_counts_df.show()
	    	# call this method to prepare top 10 hashtags DF and send them
	    	send_df_to_dashboard(hashtag_counts_df)
		except:
	    	e = sys.exc_info()[0]
	    	print("Error: %s" % e)

The last step in our Spark application is to send the hashtag_counts_df data frame to the dashboard application. So we’ll convert the data frame into two arrays, one for the hashtags and the other for their counts. Then we’ll send them to the dashboard application through the REST API.

	def send_df_to_dashboard(df):
		# extract the hashtags from dataframe and convert them into array
		top_tags = [str(t.hashtag) for t in df.select("hashtag").collect()]
		# extract the counts from dataframe and convert them into array
		tags_count = [p.hashtag_count for p in df.select("hashtag_count").collect()]
		# initialize and send the data through REST API
		url = 'http://localhost:5001/updateData'
		request_data = {'label': str(top_tags), 'data': str(tags_count)}
		response = requests.post(url, data=request_data)


## Resources

[https://www.toptal.com/apache/apache-spark-streaming-twitter](https://www.toptal.com/apache/apache-spark-streaming-twitter)