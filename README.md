Mongo spark claster
===================

Test bed.
Spark claster with mongo hadoop connector.


Roadmap
-------

* Mongo shards (2 replica sets by 3 nodes each - 1 master and 2 slave),
  current 1 replica set from 1 master and 2 slave;

* run hadoop/spark on each one slave mongo instance from replica set;

* add example submit python task in spark claster.


Run
---

```
vagrant up
```


Spark shell
-----------

Run container with shell in vagrant `vagrant ssh ubuntu-03` (slave mongo instance)

```bash
docker run --rm -it -p 8088:8088 -p 8042:8042 -h localhost --net host evgeniyklemin/spark-mongodb bash
```


Run pyspark into slave mongo container

```bash
pyspark --jars ${SPARK_HOME}/mongo-java-driver-${MONGO_JAVA_VERSION}.jar,${SPARK_HOME}/mongo-hadoop-spark-${MONGO_HADOOP_VERSION}.jar \
--driver-class-path ${SPARK_HOME}/mongo-hadoop-spark-${MONGO_HADOOP_VERSION}.jar \
--py-files ${SPARK_HOME}/mongo-hadoop/spark/src/main/python/pymongo_spark.py,${SPARK_HOME}/mongo-hadoop/spark/src/main/python/dist/pymongo_spark-0.1.dev0-py2.6.egg
```


**Python example**

```python
import pprint
import pymongo_spark
pymongo_spark.activate()

# Disable huge logging
log4j = sc._jvm.org.apache.log4j
log4j.LogManager.getRootLogger().setLevel(log4j.Level.ERROR)

mongo_rdd_config = {
    'mongo.input.query': '{"borough": "Bronx"}',
    'mongo.input.fields': '{"borough": 1, "grades": 1}',
    'mongo.input.split.use_range_queries': 'true',
    'mongo.job.verbose': 'false'
}

mongo_rdd = sc.mongoRDD('mongodb://spark:spark@172.17.8.102:27017/test.restaurants', mongo_rdd_config)
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
