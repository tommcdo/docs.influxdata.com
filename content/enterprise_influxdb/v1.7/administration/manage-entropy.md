---
title: Manage entropy without the Anti-Entropy service
menu:
  enterprise_influxdb_1_7:
    menu: Anti-entropy service
    weight: 45
    parent: Administration
---

<dt>
To follow the procedures on this page, [Anti-Entropy service](/enterprise_influxdb/v1.7/administration/anti-entropy) must be disabled. 
</dt>

In distributed systems like InfluxDB Enterprise clusters, data is written to multiple data nodes....

- flapping
- query results differ depending on shard

The expensive part of AE is the creation and comparison of the digests.

 is expensive and forces the recompaction process, migrating data in a write-optimized format into a read-optimized format.

Overwriting everything has two consequences:

- Recompaction to read-optimize including the added missing data
- Slower queries temporarily until recompaction finished

Prerequisites

- The Anti-Entropy service must be disabled. 
  - For details on how to verify that AE is disabled, see [Anti-Entropy (AE) settings](/enterprise_influxdb/v1.7/administration/config-data-nodes/#anti-entropy-ae-settings).
- Determine which shard is impacted.

### Steps to get consistent data

#### 1. Use [`influx_inspect export`](/influxdb/v1.7/tools/influx_inspect/#export) to get a dump of data in line protocol (LP) format for all shards.

#### Overwriting 

```bash
influx_inspect export [-start <timestamp> ]


#### 2. Import the `influx_inspect import` utility to overwrite 

2.a. Replay it all back to overwrite and get a total merge
2.b. Pick one shard and copy it over the other shards

