---
title: Use "/debug/vars" HTTP endpoint to display "_internal" measurement statistics
description: Monitor InfluxDB using the /debug/vars HTTP endpoint to display "_internal" measureement statistics
menu:
  platform:
    name: Use /debug/vars HTTP endpoint
    parent: Tools for monitoring InfluxDB
    weight: 5
---

## The `/debug/vars` HTTP endpoint



## Shard information

### indexType

Displays whether the shard is using `inmem` or `tsi1` indexing.

Fixes this open issue:
https://github.com/influxdata/docs.influxdata.com/issues/1921