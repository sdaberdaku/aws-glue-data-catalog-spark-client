# aws-glue-data-catalog-spark-client

AWS Glue Data Catalog client for Apache Spark

This project provides an AWS Glue Data Catalog client implementation for Apache Spark. It allows Spark applications to
interact with the AWS Glue Data Catalog as their metastore, enabling seamless integration with AWS services.

Although AWS Glue is advertised as Hive-compatible, Apache Spark cannot use it as a metastore out-of-the-box. This
repository builds a patched version of both Hive2 and Hive3, and the AWS Glue Hive Metastore Client. The required jars
can then be added to Spark's classpath to enable AWS Glue as the metastore.

The AWS Glue metastore is enabled by setting the following Spark config:

```
spark.hive.imetastoreclient.factory.class com.amazonaws.glue.catalog.metastore.AWSGlueDataCatalogHiveClientFactory
```

Please refer to this repository for further
info: https://github.com/awslabs/aws-glue-data-catalog-client-for-apache-hive-metastore.

## Notes

The https://github.com/awslabs/aws-glue-data-catalog-client-for-apache-hive-metastore repository follows a branch-based
approach in managing different versions and has renamed/deleted older branches in the past. To avoid future issues I
have forked the original `branch-3.4.0` main branch of the repository into my GitHub account.

This project also fixes the missing `conjars` maven repository, required when building the Glue Catalog Client.

The following Apache Spark versions have been currently tested. It is important to check the compatibility of the
selected Apache Spark version with the corresponding Hive and Hadoop versions used to build it.

| Spark Version | Hive 2 Version | Hive 3 Version | Hadoop Version | Release Name |
|---------------|----------------|----------------|----------------|--------------|
| 4.1.0         | 2.3.10         | 3.1.3          | 3.4.2          | v4.1.0       |
| 4.0.1         | 2.3.10         | 3.1.3          | 3.4.1          | v4.0.1       |
| 3.5.x         | 2.3.9          | 3.1.3          | 3.3.4          | v3.5.x       |
