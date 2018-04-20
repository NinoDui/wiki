---
title: "Presto"
date: 2018-04-20 11:53
category: SQL
tag: Presto, SQL, DataAnalysis
---

[TOC]
# Presto
[Presto Document Root](https://prestodb.io/docs/current/)
Amazon Athena query engine is based on Presto 0.172.

## What is Presto

Presto is
- an open source distributed SQL query engine
- for running interactive analytic queries against data sources of all sizes ranging from gigabytes to petabytes.

## SQL

### Cast and Concat
```sql
CONCAT(str1, str2, CAST(int1 AS VARCHAR))
```
