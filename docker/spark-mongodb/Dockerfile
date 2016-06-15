FROM sequenceiq/spark:1.6.0

MAINTAINER Evgeniy Klemin "evgeniy.klemin@gmail.com"


ENV MONGO_HADOOP_VERSION 1.5.2
ENV MONGO_JAVA_VERSION 3.2.2


RUN yum install -y git python-pip gcc python-devel \
        && pip install ez_setup


# get mongo-hadoop and mongo java driver
RUN curl -s -L http://search.maven.org/remotecontent?filepath=org/mongodb/mongo-hadoop/mongo-hadoop-spark/${MONGO_HADOOP_VERSION}/mongo-hadoop-spark-${MONGO_HADOOP_VERSION}.jar > ${SPARK_HOME}/mongo-hadoop-spark-${MONGO_HADOOP_VERSION}.jar \
        && curl -s -L http://central.maven.org/maven2/org/mongodb/mongo-java-driver/3.2.2/mongo-java-driver-${MONGO_JAVA_VERSION}.jar > ${SPARK_HOME}/mongo-java-driver-${MONGO_JAVA_VERSION}.jar

# install pymongo-spark 
RUN git clone https://github.com/mongodb/mongo-hadoop.git ${SPARK_HOME}/mongo-hadoop \
        && rm -rf ${SPARK_HOME}/mongo-hadoop/.git \
        && cd ${SPARK_HOME}/mongo-hadoop/spark/src/main/python \
        && python setup.py install

ENV PYTHONPATH ${SPARK_HOME}/python:${SPARK_HOME}/mongo-hadoop/spark/src/main/python
