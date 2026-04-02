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

Launch a PySpark session with Spark 3.5 Runtime Addon and run "writeTable.py":

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


### 2. Read the Iceberg Table with SparklyR

Launch a SparklyR session with the SparklyR Community runtime provided above, and run "readTable.R":

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


## Summary and Next Steps

**Cloudera AI Runtimes** provide a structured way to package and manage reproducible environments for data science and machine learning workloads, enabling teams to define lightweight, customizable containers with specific editors, languages, and dependency stacks.

Runtimes are registered and managed in the Runtime Catalog, which lists all available standard and custom environments for interactive sessions or production workloads, and can be imported from AWS ECR and other image registries.

Administrators and data scientists can create and import new runtimes in the catalog by adding custom images and credentials, enabling secure access to registries like AWS ECR and on‑prem registries, and ensuring that the right tools and packages are available where and when teams need them. ([Cloudera Documentation][1])

### Cloudera AI Runtime Documentation & Blogs

* **Runtime Catalog documentation** – Official guide to viewing and managing the Runtime Catalog in Cloudera AI. ([Cloudera Documentation][1])
  *[https://docs.cloudera.com/machine-learning/1.5.5/runtimes/topics/ml-using-runtime-catalog.html](https://docs.cloudera.com/machine-learning/1.5.5/runtimes/topics/ml-using-runtime-catalog.html)*

* **Adding new ML Runtimes** – How to register custom ML Runtimes and add Docker registry credentials. ([Cloudera Documentation][2])
  *[https://docs.cloudera.com/machine-learning/cloud/managing-runtimes/topics/ml-adding-new-ml-runtimes.html](https://docs.cloudera.com/machine-learning/cloud/managing-runtimes/topics/ml-adding-new-ml-runtimes.html)*

* **Adding Docker registry credentials in CAI** – Instructions for adding Docker registry credentials (e.g., ECR) for pulling images. ([Cloudera Documentation][3])
  *[https://docs.cloudera.com/machine-learning/1.5.5/managing-runtimes/topics/ml-add-docker-registry-credentials-runtimes.html](https://docs.cloudera.com/machine-learning/1.5.5/managing-runtimes/topics/ml-add-docker-registry-credentials-runtimes.html)*

* **ML Runtimes overview and customization** – Full docs on ML Runtimes, editors, kernels, add‑ons, and custom images. ([Cloudera Documentation][4])
  *[https://docs.cloudera.com/machine-learning/cloud/runtimes/index.html](https://docs.cloudera.com/machine-learning/cloud/runtimes/index.html)*

* **Cloudera Blog: Building Custom Runtimes with Editors** – Example of customizing and using different editors in ML Runtimes. ([blog.cloudera.com][5])
  *[https://blog.cloudera.com/building-custom-runtimes-with-editors-in-cloudera-machine-learning/](https://blog.cloudera.com/building-custom-runtimes-with-editors-in-cloudera-machine-learning/)*
