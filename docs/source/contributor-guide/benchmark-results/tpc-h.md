<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

# Apache DataFusion Comet: Benchmarks Derived From TPC-H

The following benchmarks were performed on a Linux workstation with PCIe 5, AMD 7950X CPU (16 cores), 128 GB RAM, and
data stored locally in Parquet format on NVMe storage. Performance characteristics will vary in different environments
and we encourage you to run these benchmarks in your own environments.

The tracking issue for improving TPC-H performance is [#391](https://github.com/apache/datafusion-comet/issues/391).

![](../../_static/images/benchmark-results/0.9.0/tpch_allqueries.png)

Here is a breakdown showing relative performance of Spark and Comet for each query.

![](../../_static/images/benchmark-results/0.9.0/tpch_queries_compare.png)

The following chart shows how much Comet currently accelerates each query from the benchmark in relative terms.

![](../../_static/images/benchmark-results/0.9.0/tpch_queries_speedup_rel.png)

The following chart shows how much Comet currently accelerates each query from the benchmark in absolute terms.

![](../../_static/images/benchmark-results/0.9.0/tpch_queries_speedup_abs.png)

The raw results of these benchmarks in JSON format is available here:

- [Spark](0.9.0/spark-tpch.json)
- [Comet](0.9.0/comet-tpch.json)

# Scripts

Here are the scripts that were used to generate these results.

## Apache Spark 

```shell
#!/bin/bash
export SPARK_HOME=/opt/spark-3.5.6-bin-hadoop3/
export SPARK_MASTER=spark://woody:7077
$SPARK_HOME/bin/spark-submit \
    --master $SPARK_MASTER \
    --conf spark.driver.memory=8G \
    --conf spark.executor.instances=1 \
    --conf spark.executor.cores=8 \
    --conf spark.cores.max=8 \
    --conf spark.executor.memory=16g \
    --conf spark.memory.offHeap.enabled=false \
    --conf spark.memory.offHeap.size=8g \
    --conf spark.eventLog.enabled=true \
    tpcbench.py \
    --name spark \
    --benchmark tpch \
    --data /mnt/bigdata/tpch/sf100/ \
    --queries ../../tpch \
    --output . \
    --iterations 1
```

## Apache Spark + Comet

```shell
#!/bin/bash
export COMET_JAR=/home/andy/git/apache/datafusion-comet/spark/target/comet-spark-spark3.5_2.12-0.9.0.jar
export SPARK_HOME=/opt/spark-3.5.6-bin-hadoop3/
export SPARK_MASTER=spark://woody:7077
$SPARK_HOME/bin/spark-submit \
    --master $SPARK_MASTER \
    --jars $COMET_JAR \
    --driver-class-path $COMET_JAR \
    --conf spark.driver.memory=8G \
    --conf spark.executor.instances=1 \
    --conf spark.executor.cores=8 \
    --conf spark.cores.max=8 \
    --conf spark.executor.memory=8g \
    --conf spark.memory.offHeap.enabled=true \
    --conf spark.memory.offHeap.size=8g \
    --conf spark.sql.adaptive.enabled=true \
    --conf spark.eventLog.enabled=true \
    --conf spark.driver.extraClassPath=$COMET_JAR \
    --conf spark.executor.extraClassPath=$COMET_JAR \
    --conf spark.plugins=org.apache.spark.CometPlugin \
    --conf spark.shuffle.manager=org.apache.spark.sql.comet.execution.shuffle.CometShuffleManager \
    --conf spark.comet.exec.sortMergeJoinWithJoinFilter.enabled=false \
    --conf spark.comet.exec.replaceSortMergeJoin=true \
    --conf spark.comet.enabled=true \
    --conf spark.comet.exec.shuffle.mode=auto \
    --conf spark.comet.cast.allowIncompatible=true \
    --conf spark.comet.exec.shuffle.compression.codec=lz4 \
    --conf spark.comet.exec.shuffle.compression.level=1 \
    tpcbench.py \
    --name comet \
    --benchmark tpch \
    --data /mnt/bigdata/tpch/sf100/ \
    --queries ../../tpch \
    --output . \
    --iterations 1
```