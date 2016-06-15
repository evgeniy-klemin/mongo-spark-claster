Mongo spark claster
===================

Test bed.
Hadoop spark with mongodb data provider.


Roadmap
-------

* Mongo shards (2 replica sets by 3 nodes each - 1 master and 2 slave),
  current 1 replica set from 1 master and 2 slave;

* run hadoop/spark on each one slave mongo instance from replica set;

* add example submit python task in spark claster.


Run

```
vagrant up
```


Spark shell
-----------

Run container with shell in vagrant `vagrant ssh ubuntu-03` (slave mongo instance)

```
docker run --rm -it -p 8088:8088 -p 8042:8042 -h localhost --net host evgeniyklemin/spark-mongodb bash
```


Run pyspark into slave mongo container

```
pyspark --jars ${SPARK_HOME}/mongo-java-driver-${MONGO_JAVA_VERSION}.jar,${SPARK_HOME}/mongo-hadoop-spark-${MONGO_HADOOP_VERSION}.jar \
--driver-class-path ${SPARK_HOME}/mongo-hadoop-spark-${MONGO_HADOOP_VERSION}.jar \
--py-files ${SPARK_HOME}/mongo-hadoop/spark/src/main/python/pymongo_spark.py,${SPARK_HOME}/mongo-hadoop/spark/src/main/python/dist/pymongo_spark-0.1.dev0-py2.6.egg
```


Example

```
log4j = sc._jvm.org.apache.log4j
log4j.LogManager.getRootLogger().setLevel(log4j.Level.ERROR)

import pprint
import pymongo_spark
pymongo_spark.activate()

mongo_rdd = sc.mongoRDD('mongodb://spark:spark@172.17.8.100:27017/test.restaurants', {
    'mongo.input.query': '{"borough": "Bronx"}',
    'mongo.input.fields': '{"borough": 1, "grades": 1}',
    'mongo.input.split.use_range_queries': 'true',
    'mongo.job.verbose': 'false'
})
mongo_rdd.cache()

pprint.pprint(mongo_rdd.first())

pprint.pprint(mongo_rdd
    .map(lambda x: (x['borough'], x['grades']))
    .mapValues(lambda x: [(item['grade'], 1) for item in x])
    .flatMapValues(lambda x: x)
    .map(lambda x: ((x[0], x[1][0]), x[1][1]))
    .reduceByKey(lambda x, y: x + y)
    .sortByKey()
    .collect())

```