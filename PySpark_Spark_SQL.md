# PySpark 00X - Spark SQL

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

Python, Spark, HDFS

# How to Complete

Guided steps through DF and Spark SQL. Should be completed in Spark Shell. Will need files on HDFS to populate dataframes.


## 1. Dataframes

	df1 = spark.read.option("multiline", "true").json("file:///{file_path}/people.json")

	df1.show()

	//df2 = spark.read.text("file:///{file_path}/people.txt")

	//df2.show()

	df2 = spark.read.format("csv").option("sep","\t").load("file:///{file_path}/people.csv")

	df3 = spark.read.csv("file:///{file_path}/people.csv")

	df3.show()

	df1.printSchema()


	df1.select("name").show()

	df1.select($"age" +1).show()

	df = sc.parallelize([(4, "blah", 2),(2, "", 3),(56, "foo", 3),(100, None, 5)]).toDF(["A", "B", "C"])

	newDf = df.withColumn("D", when(df.B == None, 0).when(df.B == "", 0).otherwise(1))



### Drop duplicates


	data = sc.parallelize([("Foo",41,"US",3),

		("Foo",39,"UK",1),

		("Bar",57,"CA",2),

		("Bar",72,"CA",2),

		("Baz",22,"US",6),

		("Baz",36,"US",6)]).toDF(["x","y","z","count"])



	data.dropDuplicates(["x","count"]).show()


	data2 = sc.parallelize([(0, "hello"), (1, "world")]).toDF(["id", "text"])

	from pyspark.sql.functions import col, udf
	from pyspark.sql.types import StringType


	upper = udf(lambda x: upper(x), StringType())

	data2.withColumn("upper", upper(col("text"))).show()

	dataFrame = sc.parallelize([("10.023", "75.0125", "00650"),("12.0246", "76.4586", "00650"), ("10.023", "75.0125", "00651")]).toDF(["lat","lng", "zip"])

	dataFrame.printSchema()

	dataFrame.select("*").where(dataFrame.zip == "00650").show()


### Join Operations in spark


	emp = [(1,"Smith",-1,"2018","10","M",3000),

		(2,"Rose",1,"2010","20","M",4000),

	 	(3,"Williams",1,"2010","10","M",1000),

		(4,"Jones",2,"2005","10","F",2000),

		(5,"Brown",2,"2010","40","",-1),

		(6,"Brown",2,"2010","50","",-1)

		]

	empColumns = ["emp_id","name","superior_emp_id","year_joined",
		"emp_dept_id","gender","salary"]
	  
	  
	empDF = sc.parallelize(emp).toDF(empColumns)

	empDF.show()

	dept = [("Finance",10),

		("Marketing",20),

		("Sales",30),

		("IT",40)

		]

	deptColumns = ["dept_name","dept_id"]

	deptDF = sc.parallelize(dept).toDF(deptColumns)

	deptDF.show()

	empDF.join(deptDF,empDF.emp_dept_id == deptDF.dept_id,"inner").show()

	empDF.join(deptDF,empDF.emp_dept_id == deptDF.dept_id,"outer").show()

	empDF.join(deptDF,empDF.emp_dept_id == deptDF.dept_id,"full").show()

	empDF.join(deptDF,empDF.emp_dept_id == deptDF.dept_id,"fullouter").show()



### Spark aggregate 


	simpleData = [("James","Sales","NY",90000,34,10000),

		("Michael","Sales","NY",86000,56,20000),

		("Robert","Sales","CA",81000,30,23000),

		("Maria","Finance","CA",90000,24,23000),

		("Raman","Finance","CA",99000,40,24000),

		("Scott","Finance","NY",83000,36,19000),

		("Jen","Finance","NY",79000,53,15000),

		("Jeff","Marketing","CA",80000,25,18000),

		("Kumar","Marketing","NY",91000,50,21000)

	]

	df = sc.parallelize(simpleData).toDF(["employee_name","department","state","salary","age","bonus"])

	df.show()



	df.groupBy("department").count().show()

	df.groupBy("department").avg("salary").show()

	df.groupBy("department").sum("salary").show()

	df.groupBy("department").min("salary").show()

	df.groupBy("department").max("salary").show()

	df.groupBy("department").mean("salary").show()




	df.groupBy("department","state").sum("salary","bonus").show()

	df.groupBy("department","state").avg("salary","bonus").show()

	df.groupBy("department","state").max("salary","bonus").show()

	df.groupBy("department","state").min("salary","bonus").show()

	df.groupBy("department","state").mean("salary","bonus").show()

	df.groupBy("department","state").sum("salary","bonus").show()

	from pyspark.sql.functions import sum,avg,max,count

	df.groupBy("department").agg(sum("salary").alias("sum_salary"),avg("salary").alias("avg_salary"),sum("bonus").alias("sum_bonus"),max("bonus").alias("max_bonus")).show()


## 2. Using SparkSQL

	df = spark.read.option("multiline", "true").json("{file_path}/people.json")

	df.show()

	df.createOrReplaceTempView("people")

	sqlDF = spark.sql("SELECT * FROM people")

	sqlDF.show()




---------------------------------------------------------------------------------------------
## 3. Generic Load Functions

	empdf1 = spark.read.option("multiline", "true").json("{file_path}/people.json")

	empdf1.write.parquet("{file_path}/employee_101235.parquet")


	parquetfiledf = spark.read.parquet("{file_path}/employee_101235.parquet")  

	parquetfiledf.createOrReplaceTempView("parquetFile")

	namedf = spark.sql("SELECT * FROM parquetFile")

	namedf.show()


	peopleDF = spark.read.format("json").option("multiline", "true").load("{file_path}/people.json")

	peopleDF.select("name", "age").write.format("parquet").save("{file_path}/namesAndAges.parquet")


	peopleDF = spark.read.format("json").option("multiline", "true").format("json").load("{file_path}/people.json")

	peopleDF.write.partitionBy("name").format("parquet").save("{file_path}/nameAndAgesPartitioned.parquet") 


-----------------------------------------------------------------------------------
## 4. Global Temporary View
//Temporary views in Spark SQL are session-scoped and will disappear if the session that creates it terminates. If you want to have a temporary view that is shared among all sessions and keep alive until the Spark application terminates, you can create a global temporary view. Global temporary view is tied to a system preserved database global_temp, and we must use the qualified name to refer it, e.g. SELECT * FROM global_temp.view1.

	df.createGlobalTempView("people6")


	// Global temporary view is tied to a system preserved database `global_temp`

	spark.sql("SELECT * FROM global_temp.people6").show()

	// Global temporary view is cross-session

	spark.newSession().sql("SELECT * FROM global_temp.people6").show()