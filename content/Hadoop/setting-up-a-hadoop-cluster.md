---
title: "Setting Up a Hadoop Cluster"
date: 2018-04-18 19:54
category: BigData
tag: Hadoop, Configuration
---

[TOC]

# Setting up a Hadoop Cluster

## Cluster Specification
### Cluster Sizing (to be added)

### Network Topology
A common Hadoop cluster architecture consists of a two-level network topology (Rack + Switch)
#### Rack Awareness
##### Run on Single Rack
Nothing to do, default.
##### Run on multrirack clusters
Network locations (nodes and racks)
- represented in a tree
- reflects the network “distance” between locations.
- The Hadoop configuration **must specify a map between node addresses and network locations**.
```Java
public interface DNSToSwitchMapping {
    public List<String> resolve(List<String> names);
}
```
The `net.topology.node.switch.mapping.impl` configuration property defines an implementation of the `DNSToSwitchMapping` interface that the namenode and the resource manager use to **resolve worker node network locations**.
- Default implementation is `ScriptBasedMapping`, which runs a **user-defined script** to determine the mapping. Most installations don’t need to implement the interface themselves.
- The script’s location is controlled by the property `net.topology.script.file.name`.
- If **no script location** is specified, the default behavior is to **map all nodes to a single network location**, called */default-rack*.

## Cluster Setup and Installation

### Installing Java (Skip)
### Creating Unix User Accounts
The HDFS, MapReduce, and YARN services are usually run as separate users, named `hdfs`, `mapred`, and `yarn`, respectively. They all belong to the same `hadoop` group.
### Installing Hadoop (Skip)
### Configuring SSH
Cluster-wide operations performed by:
- Hadoop control scripts
- Distributed Shell
- Dedicated Hadoop management applications
```shell
cat ~/.ssh/<new-key>.pub >> ~/.ssh/authorized_keys
```
### Configuring Hadoop(Later in Detail)
### Formatting the HDFS FileSystem
Before it can be used, a brand-new HDFS installation needs to be formatted.
- Creates an empty filesystem by creating the storage directories and the initial versions of the namenode’s persistent data structures.
- **Datanodes are not involved in the initial formatting process**, since the namenode manages all of the filesystem’s metadata, and datanodes can join or leave the cluster dynamically.
- **Don’t need to say how large a filesystem to create**, since this is determined by the number of datanodes in the cluster, which can be increased as needed, long after the filesystem is formatted.
```Shell
hdfs namenode -format
```
### Starting and Stopping the Daemons
Hadoop comes with **scripts** (in *sbin*) across the whole cluster for
- running commands
- starting and stopping daemons

To use these scripts, we need to tell Hadoop which machines are in the cluster.
- `slaves`, file for this purpose, which contains a list of the machine hostnames or IP addresses, one per line.
- lists the machines that the datanodes and node managers should run on
- resides in Hadoop’s configuration directory
- may be replaced by `HADOOP_SLAVES` in `hadoop-env.sh`
- **does not need to be distributed to worker nodes**, since they are used only by the control scripts running on the namenode or resource manager.

```shell
# Start the HDFS daemon
start-dfs.sh

# find the namenode's hostname from fs.defaultFS
hdfs getconf -namenodes

# Start YARN
start-yarn.sh
```
These scripts start and stop Hadoop daemons using the `hadoop-daemon.sh` script (or the yarn-daemon.sh script, in the case of YARN).
- If you use the aforementioned scripts, you **shouldn’t** call `hadoop-daemon.sh` directly.
- If you need to control Hadoop daemons from another system or from your own scripts, use `hadoop-daemon.sh` script.

### Creating User directories
```shell
hadoop fs -mkdir /user/username
hadoop fs -chown username:username /suer/username

# Optional
hdfs dfsadmin -setSpaceQuota 1t /user/username
```

## Hadoop Configuration

- `hadoop-env.sh` + `mapred-env.sh` + `yarn-env.sh`
    - bash script
    - Environment variables that are used in the scripts to run Hadoop/MapReduce/YARN
    - The last two **overrides** the former `hadoop-env.sh`
- `core-site.xml` `hdfs-site.xml` `mapred-site.xml` `yarn-site.xml`
    - Hadoop Configuration XML
    - Configuration settings for Hadoop Core/HDFS Daemons/MapReduce Daemons/YARN Daemons
- `slaves`
    - Plain text
    - A list of machines (one per line) that each run a datanode and a node manager
- `hadoop-metrics2.properties` `log4j.properties`
    - Java Properties
- `hadoop-policy.xml`
    - Configuration settings for access control lists when running Hadoop in secure mode

These files are all found in the `etc/hadoop` directory of the Hadoop distribution.
The configuration directory can be relocated to another part of the filesystems, as long as
    - daemons are started with the --config option
    - with the `HADOOP_CONF_DIR` environment variable set

### Configuration Management

Hadoop does **not** have a single, global location for configuration information.
Each Hadoop node in the cluster has its own set of configuration files, and it is up to administrators to ensure that they are kept in sync across the system. (Hadoop cluster management tools like `Cloudera Manager` and `Apache Ambari` take care of propagating changes across the cluster.)

If expand the cluster with new machines that have a different hardware specification from the existing ones, you need a different configuration, several excellent tools such as `Chef`, `Puppet`, `CFEngine`, and `Bcfg2`.

### Environment Settings
`mapred-env.sh` and `yarn-env.sh` files override the values set in `hadoop-env.sh`.

#### Java

1. `JAVA_HOME` in `hadoop-env.sh`
2. `JAVA_HOME` in shell environment variable (if not set in `hadoop-env.sh`)

#### Memory heap size
- Default, 1000 MB (1GB) to each daemon
- controlled by `HADOOP_HEAPSIZE` in `hadoop-env.sh`

#### System logfiles
- Default `$HADOOP_HOME/logs`
- controlled by `HADOOP_LOG_DIR` in `hadoop-env.sh`

```shell
export HADOOP_LOG_DIR=/var/log/hadoop
```
Each Hadoop daemon running on a machine produces two logfiles.
- The first
    - written via log4j, name ends in `.log`
    - should be the first port of call when diagnosing problems because most application log messages are written here.
    - The standard Hadoop log4j configuration uses a daily rolling file appender to rotate logfiles.
    - Old logfiles are never deleted.
- The second
    - logfile is the combined standard output and standard error log. name ends in `.out`
    - rotated only when the daemon is restarted, and only the last five logs are retained.
    - Old logfiles are suffixed with a number between 1 and 5, with 5 being the oldest file.

#### SSH Settings
To pass extra options to SSH, define the `HADOOP_SSH_OPTS` environment variable in `hadoop-env.sh`.

### Important Hadoop Daemon Properties
These properties are set in the Hadoop site files: `core-site.xml`, `hdfs-site.xml`, and `yarn-site.xml`.

To find the actual configuration of a running daemon, visit the `/conf` page on its web server. For example, http://resource-manager-host:8088/conf shows the configuration that the resource manager is running with.

#### HDFS
`fs.defaultFS`
- designate one machine as a namenode
- an HDFS filesystem URI
    - whose host is the namenode’s host‐name or IP address
    - whose port is the port that the namenode will listen on for RPCs.
- If no port is specified, the default of **8020** is used.

#### YARN
To run YARN, you need to **designate one machine as a resource manager**.
- set the property `yarn.resourcemanager.hostname` to the hostname or IP address of the machine running the resource manager.

#### Memory Settings in YARN and MapReduce (to be added)
#### CPU Settings in YARN and MapReduce (to be added)

### Hadoop Daemon Address and Ports
### Other Hadoop Properties
