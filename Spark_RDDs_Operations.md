# Spark Scala 00X - RDDs and Operations

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

Scala, Spark

# How to Complete

Guided steps through RDD creation, transformations, and actions. Should be run through the Spark shell,

## 1. RDD Parallelization

val rdd1 = sc.parallelize(Array("jan","feb"))

rdd1.collect

## 2. Map

val words = sc.parallelize(Seq("ja","feb","kiojh"))

val wordpair = words.map(w=> (w.charAt(0), w))

wordpair.collect

## 3. Zip
val a = sc.parallelize(List("dog","almon"),3)

val b = a.map(_.length)

val c = a.zip(b)

c.collect

## 4. Filter

val a = sc.parallelize(1 to 10)

val b = a.filter(_%2 ==0)

b.collect

## 5. Group By

val a = sc.parallelize(1 to 10)

a.groupBy(x=> { if(x%2 ==0) "even" else "odd"}).collect // We can call collect on the created RDD as opposed to assigning it to a variable


## 6. Key By / Join / ToDebugString

val a = sc.parallelize(List("hi","hello","world"),3)

val b = a.keyBy(_.length)

val c = sc.parallelize(List("hi","hello","koil"),3)

val d = c.keyBy(_.length)

val k = b.join(d)

k.toDebugString // To show lineage

## 7. Read from text file

val textFile = sc.textFile("/user/maria/test.csv") // location on Hive. can use local file system using "file:///{file_path}"

textFile.map(line => line.split(" ")).collect()

textFile.flatMap(line => line.split(" ")).collect()

## 8. Actions

1) val a = sc.parallelize(1 to 10)

	a.reduce(_+_)

2)  val b = sc.parallelize(List("hi","hello","hii","gbhyu","youtu","cat","well","hell","anitha"),3)

	b.takeOrdered(2)

	b.take

	b.first

3) val c = sc.parallelize(List((3,6),(3,7),(5,8),(3,"Dog")),2)

	c.countByKey

	c.countByValue

4) val t = sc.parallelize(List((1, 2), (3, 4), (3, 6)))  // Also used for 5-10

	val y = t.reduceByKey((x, y) => x + y)

	y.collect

5) t.groupByKey().collect

6) t.mapValues(x => x+1).collect

7) t.flatMapValues(x => (x to 5)).collect

8) t.keys.collect

9) t.values.collect

10) t.sortByKey().collect



