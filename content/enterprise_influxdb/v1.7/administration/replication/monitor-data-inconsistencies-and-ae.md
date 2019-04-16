---
title: Monitor shard data inconsistencies and the Anti-Entropy service
description: Manually find data inconsistencies and monitor shard insconsistencies and the AE service.
aliases:
  - /enterprise_influxdb/v1.7/guides/anti-entropy/
menu:
  enterprise_influxdb_1_7:
    menu: Anti-entropy service
    weight: 40
    parent: Entropy
---



## Use `influxd-ctl show` to find shard data inconsistencies

For information on using the `influxd-ctl show` command and detecting shard data inconsistencies, see [Use "influxd-ctl entropy" commands to  manage entropy](/enterprise_influxdb/v1.7/administration/replication/anti-entropy-command-line).

## Monitor Anti-Entropy in dashboards

[Chronograf](/chronograf/latest), the data visualization component and user interface for InfluxDB Enterprise clusters, can be used to build custom metrics and dashboards to help you monitor the Anti-Entropy service.

The [InfluxDB Enterprise Cluster Stats monitoring dashboard](/platform/monitoring/influxdata-platform/monitoring-dashboards/dashboard-enterprise-monitoring) example can be used
for monitoring the health of your InfluxDB Enterprise clusters. This example dashboard includes the following two metrics related to the Anti-Entropy service:

- [Anti-Entropy Errors](/platform/monitoring/influxdata-platform/monitoring-dashboards/dashboard-enterprise-monitoring/#anti-entropy-errors) displays the count of errors from Anti-Entropy service.
- [Anti-Entropy Jobs](/platform/monitoring/influxdata-platform/monitoring-dashboards/dashboard-enterprise-monitoring/#anti-entropy-jobs)

## Create alerts and metrics using the `_internal` Anti-Entropy measurement statistics

The [`_internal` database](https://docs.influxdata.com/platform/monitoring/influxdata-platform/tools/measurements-internal/#using-the-internal-database) captures measurement statistics related to InfluxDB Enterprise clusters. You can be use the Anti-Entropy and other measurement statistics to create custom alerts and metrics. The Anti-Entropy measurement statistics include the following:

- bytesRx: The number of bytes received by the data node.
- errors: The total number of jobs that resulted in errors
- jobs: The total number of jobs executed by the data node
- jobsActive: The number of active (currently executing) jobs

For details on these measurement statistics, see [`ae` (Anti-Entropy engine) measurement statistics](https://docs.influxdata.com/platform/monitoring/influxdata-platform/tools/measurements-internal/#ae-enterprise-only). Queries using `ae` measurements are included in the [InfluxDB Enterprise Cluster Stats monitoring dashboard](/platform/monitoring/influxdata-platform/monitoring-dashboards/dashboard-enterprise-monitoring).