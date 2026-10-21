# aws-glue-data-catalog-spark-client

Jars that let Apache Spark use the AWS Glue Data Catalog as its Hive metastore.

AWS Glue is advertised as Hive-compatible, but Spark cannot use it as a metastore out of the box. Spark's Hive client
has no way to plug in a different metastore client, and the
[AWS Glue Data Catalog client](https://github.com/awslabs/aws-glue-data-catalog-client-for-apache-hive-metastore) needs
one. This repository builds a patched Hive 2 that adds that option ([HIVE-12679](https://issues.apache.org/jira/browse/HIVE-12679))
and the Glue client itself, for each Spark version, and publishes the jars as GitHub releases.

There is no Java source here: the repository is the build recipe in `.github/workflows/build.yaml`. The build and the
reasons for it are described in the articles
[Building a custom Apache Spark Docker image with AWS Glue Data Catalog support as metastore](https://medium.com/towards-data-engineering/building-a-custom-apache-spark-docker-image-with-aws-glue-data-catalog-support-as-metastore-95ef9a709794)
and [Enabling AWS Glue support in Apache Spark 4](https://medium.com/towards-data-engineering/enabling-aws-glue-support-in-apache-spark-4-2c1fca7a7149).

## Usage

Each release, tagged after the Spark version (for example `v4.2.0`), has three jars:

- `hive-common-<hive>.jar` and `hive-exec-<hive>-core.jar`: the patched Hive 2 jars. They replace the files with the
  same names in `$SPARK_HOME/jars`.
- `aws-glue-datacatalog-spark-client.jar`: the Glue client. Add it to `$SPARK_HOME/jars`.

The Glue client also needs the AWS SDK for Java 1.x jars `aws-java-sdk-glue`, `aws-java-sdk-core` and
`aws-java-sdk-sts` (1.12.x, from Maven Central). For Spark 4.2.0:

```bash
RELEASE=https://github.com/sdaberdaku/aws-glue-data-catalog-spark-client/releases/download/v4.2.0
wget -O "$SPARK_HOME/jars/hive-common-2.3.10.jar" "$RELEASE/hive-common-2.3.10.jar"
wget -O "$SPARK_HOME/jars/hive-exec-2.3.10-core.jar" "$RELEASE/hive-exec-2.3.10-core.jar"
wget -P "$SPARK_HOME/jars/" "$RELEASE/aws-glue-datacatalog-spark-client.jar"
```

Then enable Hive support and set the Glue client factory:

```
spark.sql.catalogImplementation hive
spark.hive.imetastoreclient.factory.class com.amazonaws.glue.catalog.metastore.AWSGlueDataCatalogHiveClientFactory
spark.hadoop.aws.region eu-central-1
```

The client uses the default AWS credentials chain. [spark-with-glue](https://github.com/sdaberdaku/spark-with-glue)
publishes Spark images with these jars already installed.

## Versions

Pick the release that matches your Spark version. The Hive 2 version is the one Spark bundles.

| Spark Version | Hive 2 Version | Hive 3 Version | Hadoop Version | Release Name |
|---------------|----------------|----------------|----------------|--------------|
| 4.2.0         | 2.3.10         | 3.1.3          | 3.5.0          | v4.2.0       |
| 4.1.3         | 2.3.10         | 3.1.3          | 3.4.2          | v4.1.3       |
| 4.1.2         | 2.3.10         | 3.1.3          | 3.4.2          | v4.1.2       |
| 4.1.1         | 2.3.10         | 3.1.3          | 3.4.2          | v4.1.1       |
| 4.1.0         | 2.3.10         | 3.1.3          | 3.4.2          | v4.1.0       |
| 4.0.2         | 2.3.10         | 3.1.3          | 3.4.1          | v4.0.2       |
| 4.0.1         | 2.3.10         | 3.1.3          | 3.4.1          | v4.0.1       |
| 3.5.x         | 2.3.9          | 3.1.3          | 3.3.4          | v3.5.x       |

Spark 4.2.0 ships Hadoop 3.5.0, whose jars target Java 17. The Glue client is built with JDK 8, so it still compiles
against Hadoop 3.4.2. Hadoop is a `provided` dependency: at runtime the client uses the Hadoop bundled with Spark.

## How the jars are built

Pushing a tag starts the workflow, which runs on JDK 8 and Maven 3.9.9 and builds, in order:

1. Hive 2: the Apache Hive release with `HIVE-12679.branch-2.3.patch` from this repository.
2. Hive 3: the Apache Hive 3.1 release with the `branch_3.1.patch` from the Glue client repository. The Glue client
   build depends on it, but its jars are not released.
3. The Glue client, from [a fork](https://github.com/sdaberdaku/aws-glue-data-catalog-client-for-apache-hive-metastore)
   of its `branch-3.4.0`. The upstream repository keeps one branch per version and has renamed or deleted branches in
   the past, so the fork keeps the build reproducible.

The `.mvn/settings.xml` file points the `conjars` repository, which no longer exists, to a mirror, so that the old Hive
dependencies still resolve.

To add a Spark version, set `HIVE2_VERSION` in the workflow to the `hive.version` of that Spark release, add a row to
the table above, commit and push a tag named after the Spark version.

## License

[Apache License 2.0](LICENSE)
