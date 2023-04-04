# Spark Scala 00X - Spark SQL

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

Scala, Spark, HDFS

# How to Complete

Guided steps through DF and Spark SQL. Should be completed in Spark Shell. Will need files on HDFS to populate dataframes.


## 1. Dataframes

	var df1 = spark.read.option("multiline", "true").json("/user/will/people.json")

	df1.show()

	//var df2 = spark.read.text("/user/will/people.txt")

	//df2.show()

	var df2=spark.read.format("csv").option("sep","\t").load("/user/will/people.csv")

	var df3 = spark.read.csv("/user/will/people.csv")

	df3.show()

	df1.printSchema()


	df1.select("name").show()

	df1.select($"age" +1).show()

	val df = sc.parallelize(Seq((4, "blah", 2),(2, "", 3),(56, "foo", 3),(100, null, 5))).toDF("A", "B", "C")

	val newDf = df.withColumn("D", when($"B".isNull or $"B" === "", 0).otherwise(1))


	val df4 = Seq((1,2)).toDF("x","y")

	val myExpression = "x+y"

	//import org.apache.spark.sql.functions.expr

	df4.withColumn("z",expr(myExpression)).show()

	<br>
	//import spark.implicits._

	    val df5 = Seq(

	      ("steak", 1, 1, 150),

	      ("steak", 2, 2, 180),

	      ("fish", 3, 3, 100)

	    ).toDF("C1", "C2", "C3", "C4")

	//import org.apache.spark.sql.functions._

	df5.withColumn("C5", expr("C2/(C3 + C4)")).show()
	<br>

	//Spark column string replace when present in other column (row)

	val df6 = spark.createDataFrame(Seq(

	  ("Hi I heard about Spark", "Spark"),

	  ("I wish Java could use case classes", "Java"),

	  ("Logistic regression models are neat", "models")

	)).toDF("sentence", "label")


	val replace = udf((data: String , rep : String)=>data.replaceAll(rep, ""))

	val res = df6.withColumn("sentence_without_label", replace($"sentence" , $"label"))

	res.show()


### Drop duplicates


	val data = sc.parallelize(List(("Foo",41,"US",3),

		("Foo",39,"UK",1),

		("Bar",57,"CA",2),

		("Bar",72,"CA",2),

		("Baz",22,"US",6),

		("Baz",36,"US",6))).toDF("x","y","z","count")



	data.dropDuplicates(Array("x","count")).show()


	val dataset = Seq((0, "hello"), (1, "world")).toDF("id", "text")

	val upper: String => String = _.toUpperCase


	//import org.apache.spark.sql.functions.udf

	val upperUDF = udf(upper)

	dataset.withColumn("upper", upperUDF('text)).show

	val dataFrame = Seq(("10.023", "75.0125", "00650"),("12.0246", "76.4586", "00650"), ("10.023", "75.0125", "00651")).toDF("lat","lng", "zip")

	dataFrame.printSchema()

	dataFrame.select("*").where(dataFrame("zip") === "00650").show()


### Join Operations in spark


	val emp = Seq((1,"Smith",-1,"2018","10","M",3000),

		(2,"Rose",1,"2010","20","M",4000),

	 	(3,"Williams",1,"2010","10","M",1000),

		(4,"Jones",2,"2005","10","F",2000),

		(5,"Brown",2,"2010","40","",-1),

		(6,"Brown",2,"2010","50","",-1)

		)

	val empColumns = Seq("emp_id","name","superior_emp_id","year_joined",
		"emp_dept_id","gender","salary")
	  

	//import spark.sqlContext.implicits._
	  
	val empDF = emp.toDF(empColumns:_*)

		empDF.show()

	val dept = Seq(("Finance",10),

		("Marketing",20),

		("Sales",30),

		("IT",40)

		)

	val deptColumns = Seq("dept_name","dept_id")

	val deptDF = dept.toDF(deptColumns:_*)

	deptDF.show()

	empDF.join(deptDF,empDF("emp_dept_id") ===  deptDF("dept_id"),"inner").show()

	empDF.join(deptDF,empDF("emp_dept_id") ===  deptDF("dept_id"),"outer").show()

	empDF.join(deptDF,empDF("emp_dept_id") ===  deptDF("dept_id"),"full").show()

	empDF.join(deptDF,empDF("emp_dept_id") ===  deptDF("dept_id"),"fullouter").show()



### Spark aggregate 

	//import spark.implicits._

	val simpleData = Seq(("James","Sales","NY",90000,34,10000),

		("Michael","Sales","NY",86000,56,20000),

		("Robert","Sales","CA",81000,30,23000),

		("Maria","Finance","CA",90000,24,23000),

		("Raman","Finance","CA",99000,40,24000),

		("Scott","Finance","NY",83000,36,19000),

		("Jen","Finance","NY",79000,53,15000),

		("Jeff","Marketing","CA",80000,25,18000),

		("Kumar","Marketing","NY",91000,50,21000)

	)

	val df = simpleData.toDF("employee_name","department","state","salary","age","bonus")

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



	df.groupBy("department").agg(sum("salary").as("sum_salary"),avg("salary").as("avg_salary"),sum("bonus").as("sum_bonus"),max("bonus").as("max_bonus")).show()

	df.groupBy("department").agg(sum("salary").as("sum_salary"),avg("salary").as("avg_salary"),sum("bonus").as("sum_bonus"),stddev("bonus").as("stddev_bonus")).where(col("sum_bonus") > 50000).show()

## 2. Using SparkSQL

	val df = spark.read.option("multiline", "true").json("/user/will/people.json")

	df.show()

	df.createOrReplaceTempView("people")

	val sqlDF = spark.sql("SELECT * FROM people")

	sqlDF.show()




---------------------------------------------------------------------------------------------
## 3. Generic Load Functions

	val empdf1 = spark.read.option("multiline", "true").json("/user/will/people.json")

	empdf1.write.parquet("/user/will/employee_101235.parquet")


	val parquetfiledf = spark.read.parquet("/user/will/employee_101235.parquet")  //Look for Data in  HDFS

	parquetfiledf.createOrReplaceTempView("parquetFile")

	val namedf=spark.sql("SELECT * FROM parquetFile")

	namedf.show()


	val peopleDF = spark.read.format("json").option("multiline", "true").load("/user/will/people.json")

	peopleDF.select("name", "age").write.format("parquet").save("/user/will/namesAndAges.parquet")


	val peopleDF = spark.read.format("json").option("multiline", "true").format("json").load("/user/will/people.json")

	peopleDF.write.partitionBy("name").format("parquet").save("/user/will/nameAndAgesPartitioned.parquet") //or whatever the given name is below

	hdfs dfs -cat /user/will/employee_101234.parquet/part-00000-e4605471-4ac7-4621-9ca2-839f24cd08b7-c000.snappy.parquet


-----------------------------------------------------------------------------------
## 4. Global Temporary View
//Temporary views in Spark SQL are session-scoped and will disappear if the session that creates it terminates. If you want to have a temporary view that is shared among all sessions and keep alive until the Spark application terminates, you can create a global temporary view. Global temporary view is tied to a system preserved database global_temp, and we must use the qualified name to refer it, e.g. SELECT * FROM global_temp.view1.

	df.createGlobalTempView("people6")


	// Global temporary view is tied to a system preserved database `global_temp`

	spark.sql("SELECT * FROM global_temp.people6").show()

	// Global temporary view is cross-session

	spark.newSession().sql("SELECT * FROM global_temp.people6").show()