---
title: Manage entropy without the Anti-Entropy service
menu:
  enterprise_influxdb_1_7:
    menu: Anti-entropy service
    weight: 45
    parent: Administration
---

<dt>
The procedures on this page assume that the [Anti-Entropy service](/enterprise_influxdb/v1.7/administration/anti-entropy) is disabled. 
</dt>

## Shard entropy overview

Data is written to InfluxDB Enterprise clusters and replicated in multiple shards (copies of data) across data nodes in the cluster. The number of shard copies is determined by the replication factor (RF) and during normal operations, data is written to a shard and replicated to the shard copies without isssues. At any instant of time, the multiple copies are not exact copies of each other, but as the data is replicated among the shards, the data is eventually consistent and are replicas of each other. Due to unanticipated events, however, the copies of data between the shards can become inconsistent, and the differences between the copies need to be reconciled.

In InfluxDB Enterprise clusters, the [Anti-Entropy service](/enterprise_influxdb/v1.7/administration/anti-entropy) is intended to be used for automatically tracking and maintaining the consistency of the data copies. Depending on your application, especially when the time series data includes a massive number of time ranges (tens of thousands), you might not be able to use the Anti-Entropy service without encountering serious performance issues.

This topic provides guidance on how to manage shard entropy (the inconsistency among shards in a shard group) due to missing missing data.

## Indicators of shard entropy

When shards become inconsistent, you might notice the following:

- Inconsistent results for the same query.
- Counts of data between shards differ.
- Flapping metrics
  - Rapid alert status changes.
    - Alerts that fire and clear repeatedly in a short time period.
    - Values in graphs change erratically or switch rapidly.
- Shard copies are not the same size.

## Causes of shard entropy

Shard entropy, or inconcistencies of data between shards within a shard group, can occur when unanticipated events occur. 
Some of the most frequent causes of entropy in InfluxDB Enterprise clusters include:

- Hardware failure
  - Server crashes, particularly 
  - Power interruptions or failures
  - Network interruptions or failures
- Server crashes
  - Data node panics when inflight data could be dropped
  - When a node crashes after the write was replicated, but before the `fsync` operation on the remote node, 
- Dropping or deleting measurements
- Dropping or deleting series
- [Hinted Handoff (HH)]() overflows
- Batch write interruptions resulting in partial data writes

Overwriting everything has two consequences:

- Recompaction to read-optimize including the added missing data
- Slower queries temporarily until recompaction finished

## Manage shard entropy without the Anti-Entropy service

If you have determined that

Prerequisites

- The Anti-Entropy service must be disabled.
  - Anti-Entropy is disabled by default.
  - For details on how to verify that AE is disabled, see [Anti-Entropy (AE) settings](/enterprise_influxdb/v1.7/administration/config-data-nodes/#anti-entropy-ae-settings).
- Determine which shard is impacted.
  - When repairing missing data in a shard, choose one of two options
    - 

### Steps to get consistent data

#### 1. Use [`influx_inspect export`](/influxdb/v1.7/tools/influx_inspect/#export) to get a dump of data in line protocol (LP) format for all shards.

For details on the `influx_inspect export` command options, 
see [export](https://docs.influxdata.com/influxdb/v1.7/tools/shell/#import-data-from-a-file-with-import)


##### Export the data from source replica shard

If the missing data includes the time ranges included in the entire shard,
you can export the data from the entire shard.

To export all of the data, run the following commmand on the data
node with less entropy. In the following example, the `export` command

```bash
influx_inspect export -database my_db -o -compress full_shard.gz]
```

##### Export data for a smaller time period

If the time range during which data was lost is significantly shorter 
than that of the entire shard, you can restrict the data export by specifying
the time range during the export.

To export a subset of data, run the following command on the data
node with less entropy, substituting starting and ending timestamps 
that cover the time period you want to fill in.

```bash
influx_inspect export -database <db_name> [-start <timestamp> -end <timestamp> ]
```


#### 2. Copy data into the target replica shard

On the data node you are repairing, use the `influxd-ctl copy-shard` command
to copy the data in the 

```bash
influxd-ctl copy-shard -database <db_name> ]
```



For details on using `influx -import`, see [Import data from a file with `-import`](https://docs.influxdata.com/influxdb/v1.7/tools/shell/#import-data-from-a-file-with-import)

2.a. Replay it all back to overwrite and get a total merge
2.b. Pick one shard and copy it over the other shards

