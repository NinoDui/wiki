---
title: "Setting Up a Hadoop Cluster"
date: 2018-04-18 19:54
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
    - The last two **overrides the former `hadoop-env.sh`
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
- The configuration directory can be relocated to another part of the filesystems, as long as 
    - daemons are started with the --config option 
    - with the `HADOOP_CONF_DIR` environment variable set

### Configuration Management

