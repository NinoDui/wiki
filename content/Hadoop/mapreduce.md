---
title: "MapReduce"
date: 2018-04-19 18:48
category: BigData
tag: Hadoop, Configuration
---

# MapReduce

[TOC]

> MapReduce is a programming model for data processing.

## Analyzing the Data With Hadoop

### Map and Reduce

MapReduce works by
- breaking the processing into `map` phase and `reduce` phase.
- Each phase has **key-value pairs as input and output**, their types chosen by the programmer.
- The map/reduce functions specified by programmer.
- The output from the map function is processed by the MapReduce framework before being sent to the reduce function. This processing **sorts and groups the key-value pairs by key**.

```Java
public class MyMapper extends Mapper<InKey, InValue, OutKey, OutValue> {

}
```

When we run this job on a Hadoop cluster, we will **package the code into a JAR file** (which Hadoop will distribute around the cluster).
Rather than explicitly specifying the name of the JAR file, we can pass a class in the Job’s `setJarByClass()` method, which Hadoop will use to locate the relevant JAR file by looking for the JAR file containing this class.

The **Output Path directory shouldn’t exist** before running the job because Hadoop will complain and not run the job.

### Java MapReduce

## Scaling Out

## Hadoop Streaming
