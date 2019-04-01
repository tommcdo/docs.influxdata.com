---
title: Manage entropy without the Anti-Entropy service
menu:
  enterprise_influxdb_1_7:
    menu: Anti-entropy service
    weight: 45
    parent: Administration
---

<dt>
The procedures on this page assume that the [Anti-Entropy service](/enterprise_influxdb/v1.7/administration/anti-entropy)
is disabled. 
</dt>

In InfluxDB Enterprise clusters, data is written to two or more replica shards 

- flapping
- query results differ depending on shard

## Causes of inconsistent data between replica shards

Entropy, or data inconcistencies, tend to occur when unanticipated events occur.
Here are the most frequent causes of entropy in InfluxDB Enterprise clusters:

- Dropping or deleting measurements
- Dropping or deleting series
- Hinted Handoff (HH) overflows
- Server crashes, particularly data node panics when in-flight data could be dropped


Overwriting everything has two consequences:

- Recompaction to read-optimize including the added missing data
- Slower queries temporarily until recompaction finished

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

