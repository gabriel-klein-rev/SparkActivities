# PySpark 00X - RDDs and Operations

### Contributors:
Gabriel Klein
{ Reviewers }

## Prerequisites

Python, Spark

# How to Complete

Guided steps through RDD creation, transformations, and actions. Should be run through the Spark shell.

## 1. RDD Parallelization

	rdd1 = sc.parallelize(["jan", "feb"])

	rdd1.collect()

## 2. Map

	words = sc.parallelize(['jan','feb','kiojh'])

	wordpair = words.map(lambda w: (w[0], w))

	wordpair.collect()

## 3. Zip
	a = sc.parallelize(["dog","almon"])

	b = a.map(lambda x: len(x))

	c = a.zip(b)

	c.collect()

## 4. Filter

	a = sc.parallelize(range(1, 10))

	b = a.filter(lambda x: x%2 == 0)

	b.collect

## 5. Group By

	a = sc.parallelize(range(1, 10))

	b = a.groupBy(lambda x: "even" if x%2 == 0 else "odd")

	b.collect() // Notice the output

	list(b.collect()[0][1]) // To easily read the accompanying iterable

## 6. Key By / Join / ToDebugString

	a = sc.parallelize(["hi","hello","world"])

	b = a.keyBy(lambda x: len(x))

	c = sc.parallelize(["hi","hello","koil"])

	d = c.keyBy(lambda x: len(x))

	k = b.join(d)

	k.collect()

	k.toDebugString() // To show lineage

## 7. Read from text file

	filename = "file:///user/{username}/file.csv" // Edit to fit file path

	a = sc.textFile(filename) 

	a.map(lambda line: line.split(" ")).collect()

	a.flatMap(lambda line: line.split(" ")).collect()

## 8. Actions

1) from operator import add

	a = sc.parallelize(range(1,10))

	a.reduce(add)

2)  b = sc.parallelize(["hi","hello","hii","gbhyu","youtu","cat","well","hell","anitha"])

	b.takeOrdered(2)

	b.take(5)

	b.first()

3) c = sc.parallelize([(3,6),(3,7),(5,8),(3,"Dog")])

	c.countByKey()

	c.countByValue()

4) t = sc.parallelize([(1, 2), (3, 4), (3, 6)])  // Also used for 5-10

	y = t.reduceByKey(lambda x, y: x + y)

	y.collect()

5) t.groupByKey().collect

6) t.mapValues(lambda x: x+1).collect()

7) t.flatMapValues(x => (x to 5)).collect()

8) t.keys().collect()

9) t.values().collect()

10) t.sortByKey().collect()



