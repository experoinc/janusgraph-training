# Running JanusGraph OLAP on YARN

## Setup

From your JanusGraph installation directory, perform the following steps.
1. Create a `hadoop_conf` directory.
1. Copy `mapred-site.xml` and `yarn-site.xml` from your Hadoop cluster into `hadoop_conf`.
1. Copy the `janusgraph-spark-0.5.3.jar` jar into `hadoop_conf`.
1. Export a new env var: `export HADOOP_CONF=hadoop_conf`.
1. Append `$HADOOP_CONF` to your `$CLASSPATH`: `export CLASSPATH=$CLASSPATH:$HADOOP_CONF`

Setup a new OLAP properties file: `vi conf/hadoop-graph/read-yarn.properties`.
Copy the following in as its contents:

```properties
#
# Hadoop Graph Configuration
#
gremlin.graph=org.apache.tinkerpop.gremlin.hadoop.structure.HadoopGraph
gremlin.hadoop.graphReader=org.janusgraph.hadoop.formats.hbase.HBaseInputFormat
gremlin.hadoop.graphWriter=org.apache.hadoop.mapreduce.lib.output.NullOutputFormat

gremlin.hadoop.jarsInDistributedCache=true
gremlin.hadoop.inputLocation=none
gremlin.hadoop.outputLocation=output
gremlin.spark.persistContext=true

#
# JanusGraph HBase InputFormat configuration
#
janusgraphmr.ioformat.conf.storage.backend=hbase
janusgraphmr.ioformat.conf.storage.hostname=localhost
janusgraphmr.ioformat.conf.storage.hbase.table=janusgraph

#
# SparkGraphComputer Configuration
#
spark.master=yarn
spark.deploy-mode=client
spark.executor.memory=1g
# include path to Spark jars and Hadoop native libs
spark.yarn.jars=/Users/twilmes/Downloads/janusgraph-0.5.3-expero-dbs/hadoop_conf/janusgraph-spark-0.5.3.jar,/Users/twilmes/Downloads/spark-2.4.0-bin-hadoop2.7/jars/spark-yarn_2.11-2.4.0.jar
spark.yarn.am.extraJavaOptions=-Djava.library.path=/opt/cloudera/parcels/CDH/lib/hadoop/lib/native
# use Java 1.8
spark.yarn.appMasterEnv.JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
spark.executorEnv.JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
# prepend shaded dependency jar to executor classpath
spark.executor.extraClassPath=/Users/twilmes/Downloads/janusgraph-0.5.3-expero-dbs/hadoop_conf/janusgraph-spark-0.5.3.jar
spark.serializer=org.apache.spark.serializer.KryoSerializer
spark.kryo.registrator=org.apache.tinkerpop.gremlin.spark.structure.io.gryo.GryoRegistrator
```

Update the `janusgraphmr.ioformat.conf.storage.hbase.*` properties with your hbase specific settings.
The property names will be `janusgraphmr.ioformat.conf.storage.hbase.` + the property name you use in your regular HBase connection configuration.

Update the following properties to match your environment:
* spark.yarn.am.extraJavaOptions 
* spark.yarn.jars
* spark.yarn.am.extraJavaOptions
* spark.yarn.appMasterEnv.JAVA_HOME
* spark.executorEnv.JAVA_HOME
* spark.executor.extraClassPath

## Running an OLAP Query

Open the gremlin shell: `./bin/gremlin.sh` and run the following commands to run a count and verify connectivity.

```groovy
graph = GraphFactory.open('conf/hadoop-graph/read-yarn.properties')
g = graph.traversal().withComputer(SparkGraphComputer.class)
g.V().count()
```