# CAI SparklyR Iceberg

## Objective

This demo shows how to read an Iceberg table with SparklyR in CAI. The demo is divided in two parts:

1. Create the Iceberg Table with PySpark
2. Read the Iceberg Table with SparklyR

## Requirements

The following are required:

* A CAI Workbench with Spark 3.5 Runtime Addon
* A SparklyR runtime: https://github.com/pdefusco/cai_sparklyr_runtime

## Demo

### 0. Setups

Create a CAI Project with the following runtime: https://github.com/pdefusco/cai_sparklyr_runtime


### 1. Create the Iceberg Table with PySpark

Launch a PySpark session with Spark 3.5 Runtime Addon and run "writeTable.py".

```
Name: PySpark Create Table Session
Editor: PBJ Workbench
Kernel: Python 3.10
Edition: Standard
Version: 2026.01
Enable Spark: version 3.5
Enable GPU: none
Resource Profile: 2 vCPU / 4 GiB Memory
```

Code:

```
import cml.data_v1 as cmldata

CONNECTION_NAME = "se-aws-edl"
conn = cmldata.get_connection(CONNECTION_NAME)
spark = conn.get_spark_session()

# Sample usage to run query through spark
EXAMPLE_SQL_QUERY = "show databases"
spark.sql(EXAMPLE_SQL_QUERY).show()

data = [
    (1, "Alice", 34),
    (2, "Bob", 45),
    (3, "Cathy", 29)
]

columns = ["id", "name", "age"]

df = spark.createDataFrame(data, columns)

df.writeTo("spark_catalog.default.people").using("iceberg").tableProperty("write.format.default", "parquet").createOrReplace()

spark.sql("SELECT * FROM spark_catalog.default.people").show()
```

Notice the PySpark Data Connection in CAI is preconfigured with Iceberg Jars and dependencies. CAI users are automatically authenticated in the Lakehouse when a SparkSession is created.

For more Spark and Iceberg best practices with PySpark, please read this article: [Spark in CML, Recommendations for Using Spark in Cloudera Machine Learning](https://community.cloudera.com/t5/Community-Articles/Spark-in-CML-Recommendations-for-using-Spark-in-Cloudera/ta-p/372164)


### 2. Read the Iceberg Table with SparklyR

Launch a SparklyR session with the SparklyR Community runtime provided above, and run "readTable.R".

```
Name: Sparklyr Read Table Session
Editor: Workbench
Kernel: R 4.5
Edition: Community
Version: 2026.03
Enable Spark: version 3.5
Enable GPU: none
Resource Profile: 2 vCPU / 4 GiB Memory
```

Code:

```
#install.packages("sparklyr")
install.packages("dplyr")

library(sparklyr)
library(dplyr)

# Spark configuration
config <- spark_config()
config$spark.kerberos.access.hadoopFileSystems <- "s3a://goes-se-sandbox/"
config$spark.executor.cores = 1
config$spark.executor.memory = "2g"
# Iceberg integration
config$spark.sql.extensions <- "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions"
config$spark.sql.catalog.spark_catalog <- "org.apache.iceberg.spark.SparkSessionCatalog"
config$spark.hadoop.iceberg.engine.hive.enabled <- "true"
config$spark.jars <- "/opt/spark/optional-lib/iceberg-spark-runtime-3.5_2.12-1.5.2.1.24.7218.0-88.jar"

# Connect to Spark
sc <- spark_connect(config = config)


people_tbl <- spark_read_source(
  sc,
  name = "people",
  source = "iceberg",           # Iceberg source
  options = list(table = "spark_catalog.default.people")
)

people_tbl %>% head()
```

Notice that in this case the Iceberg Spark dependencies and the Spark Jars property is specified when the SparkSession is created.

In particular, the "spark.jars" Spark option is set to the value of the Spark jar located in "/opt/spark/optional-lib" folder. This folder is automatically populated when the Spark Runtime Addon is configured in the session. Checking the exact jar file name is recommended when configuring the "spark.jars" option as the exact file name could vary depending on Spark and CAI versions.


## Summary and Next Steps

In **Cloudera AI (CAI)** you can use **sparklyR** to connect R workloads to Spark 3 clusters, including custom runtimes, and then access data stored in **Apache Iceberg** tables. Cloudera’s platform integrates Iceberg as a first‑class table format for scalable analytics on cloud object stores and HDFS in its Data Lakehouse offering, with catalog integration and performance features. ([Cloudera Documentation][1])

### Useful links

* **Using Apache Iceberg with Spark (Cloudera Docs)** — Overview and examples of querying Iceberg tables with Spark. ([Cloudera Documentation][1])
  [https://www.cloudera.com/runtime/7.2.16/developing-spark-applications/topics/spark-iceberg.html](https://www.cloudera.com/runtime/7.2.16/developing-spark-applications/topics/spark-iceberg.html)

* **Apache Iceberg on Cloudera** — General information about Iceberg support in Cloudera’s Lakehouse platform. ([Cloudera][3])
  [https://www.cloudera.com/open-source/apache-iceberg.html](https://www.cloudera.com/open-source/apache-iceberg.html)

* **Cloudera AI Iceberg data connection docs** — How to connect to Iceberg data lakes from CAI. ([Cloudera Documentation][4])
  [https://docs.cloudera.com/machine-learning/cloud/import-data/topics/ml-iceberg-connection.html](https://docs.cloudera.com/machine-learning/cloud/import-data/topics/ml-iceberg-connection.html)

* **Using Spark 3 from R with sparklyR (Cloudera Docs)** — Guidance and example code to connect R to Spark 3. ([Cloudera Documentation][2])

* **Introducing Apache Iceberg in Cloudera Data Platform (Blog)** — Background on Cloudera’s Iceberg integration in CDP. ([Cloudera][5])

If you want, I can provide direct runnable examples showing how to set up your CAI Spark session and R configs to work with Iceberg in a production‑ready way.

[1]: https://docs.cloudera.com/runtime/7.2.16/developing-spark-applications/topics/spark-iceberg.html?utm_source=chatgpt.com "Using Apache Iceberg with Spark"
[2]: https://docs.cloudera.com/machine-learning/cloud/spark/topics/ml-installing-sparklyr.html?utm_source=chatgpt.com "Using Spark 3 from R"
[3]: https://www.cloudera.com/open-source/apache-iceberg.html?utm_source=chatgpt.com "Apache Iceberg | Open source | Cloudera"
[4]: https://docs.cloudera.com/machine-learning/cloud/import-data/topics/ml-iceberg-connection.html?utm_source=chatgpt.com "Create an Iceberg data connection"
[5]: https://www.cloudera.com/blog/technical/introducing-apache-iceberg-in-cloudera-data-platform.html?utm_source=chatgpt.com "Introducing Apache Iceberg in Cloudera Data Platform | Blog | Cloudera"
