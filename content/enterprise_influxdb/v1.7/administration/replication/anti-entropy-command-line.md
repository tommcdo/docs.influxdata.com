---
title: Use "influxd-ctl entropy" commands to  managing entropy
description: Use the "influxd-ctl entropy" commands to show and repair entropy.
aliases:
  - /enterprise_influxdb/v1.7/guides/anti-entropy/
menu:
  enterprise_influxdb_1_7:
    menu: Anti-entropy service
    weight: 40
    parent: Entropy
---

## `influxd-ctl entropy` command line tools for managing entropy

The `influxd-ctl entropy` command and its subcommands can be used to manage shard entropy in a cluster.
It includes the following subcommands:

### `show`

Lists shards that are in an inconsistent state and in need of repair as well as
shards currently in the repair queue.

#### Syntax

```
influxd-ctl entropy show
```

### `repair`

Queues a shard for repair.
It requires a Shard ID which is provided in the [`show`](#show) output.

#### Syntax

```
influxd-ctl entropy repair <shardID>
```

Repairing entropy in a shard is an asynchronous operation.
This command will return quickly as it only adds a shard to the repair queue.
Queuing shards for repair is idempotent.
There is no harm in making multiple requests to repair the same shard even if
it is already queued, currently being repaired, or not in need of repair.

### `kill-repair`

Removes a shard from the repair queue.
It requires a Shard ID which is provided in the [`show`](#show) output.

#### Syntax

```bash
influxd-ctl entropy kill-repair <shardID>
```

This only applies to shards in the repair queue.
It does not cancel repairs on nodes that are in the process of being repaired.
Once a repair has started, requests to cancel it are ignored.

> Stopping a entropy repair for a **missing** shard operation is not currently supported.
> It may be possible to stop repairs for missing shards with the
> [`influxd-ctl kill-copy-shard`](/enterprise_influxdb/v1.7/administration/cluster-commands/#kill-copy-shard) command.

## Use case and examples

The `influxd-ctl entropy` command can be used to:

- [Detect and repair entropy](#detect-and-repair-entropy)
- [Replace an unresponsive data node](#replace-an-unresponsive-data-node)
- [Replace a data node server](#replace-a-data-node-server)
- [Repair entropy in active shards](#repair-entropy-in-active-shards)

### Detect and repair entropy

If you are seeing [indicators for shard data inconsistency](/enterprise_influxdb/v1.7/administration/entropy/manage-entropy), 
or as a step in your regular maintenance procedure, you can run the `influxd-ctl entropy show` subcommand to check whether a data node
has significant entropy that needs to be repaired.

In this example, the `influxd-ctl entropy show` subcommand are used to detect and repair entropy.

#### 1. Detect entropy

In the following example, the `influxd-ctl entropy show` sucommand is used to ist all shards and their entropy statuses.

```bash
influxd-ctl entropy show

Entropy
==========
ID     Database  Retention Policy  Start                          End                            Expires                        Status
21179  statsdb   1hour             2017-10-09 00:00:00 +0000 UTC  2017-10-16 00:00:00 +0000 UTC  2018-10-22 00:00:00 +0000 UTC  diff
25165  statsdb   1hour             2017-11-20 00:00:00 +0000 UTC  2017-11-27 00:00:00 +0000 UTC  2018-12-03 00:00:00 +0000 UTC  diff
```

#### 2. Repair the shard entropy

In the following example, the `influxd-ctl entropy repair` subcommand is used to add the twp shards (`21179` and `25165`) above
to the repair queue of the Anti-Entropy service. When successful, a message is returned indicating that the shard has been added
to the repair queue.

```bash
$ influxd-ctl entropy repair 21179

Repair Shard 21179 queued

$ influxd-ctl entropy repair 25165

Repair Shard 25165 queued
```

After you have added the shards to the repair queue, you can check on the status of the shards using
the `influxd-ctl entropy show` subcommand again. In the example, you can see that the two shards still
have a `Status` of `diff` and that a list of the queued shards appears.

```bash
influxd-ctl entropy show

Entropy
==========
ID     Database  Retention Policy  Start                          End                            Expires                        Status
21179  statsdb   1hour             2017-10-09 00:00:00 +0000 UTC  2017-10-16 00:00:00 +0000 UTC  2018-10-22 00:00:00 +0000 UTC  diff
25165  statsdb   1hour             2017-11-20 00:00:00 +0000 UTC  2017-11-27 00:00:00 +0000 UTC  2018-12-03 00:00:00 +0000 UTC  diff

Queued Shards: [21179 25165]
```

### Replace an unresponsive data node

If a data node suddenly disappears due to a catastrophic hardware failure or for any other reason, as soon as a new data node is online, the anti-entropy service will copy the correct shards to the new replacement node. The time it takes for the copying to complete is determined by the number of shards to be copied and how much data is stored in each.

For information on replacing data nodes in your InfluxDB Enterprise cluster, see [Replacing data nodes](/enterprise_influxdb/v1.7/guides/replacing-nodes/#replacing-data-nodes-in-an-influxdb-enterprise-cluster).

### Replace a data node server

Perhaps you are replacing a machine that is being decommissioned, upgrading hardware, or something else entirely.
The anti-entropy service will automatically copy shards to the new machines.

Once you have successfully run the `influxd-ctl update-data` command, you are free
to shut down the retired node without causing any interruption to the cluster.
The anti-entropy process will continue copying the appropriate shards from the
remaining replicas in the cluster.

### Repair entropy in active shards

In rare cases, the currently active shard, or the shard to which new data is
currently being written, may find itself with inconsistent data.
Because the AE process can't write to hot shards, you must stop writes to the new
shard using the [`influxd-ctl truncate-shards` command](/enterprise_influxdb/v1.7/administration/cluster-commands/#truncate-shards),
then add the inconsistent shard to the entropy repair queue:

```bash
# Truncate hot shards
influxd-ctl truncate-shards

# Show shards with entropy
influxd-ctl entropy show

Entropy
==========
ID     Database  Retention Policy  Start                          End                            Expires                        Status
21179  statsdb   1hour             2018-06-06 12:00:00 +0000 UTC  2018-06-06 23:44:12 +0000 UTC  2018-12-06 00:00:00 +0000 UTC  diff

# Add the inconsistent shard to the repair queue
influxd-ctl entropy repair 21179
```