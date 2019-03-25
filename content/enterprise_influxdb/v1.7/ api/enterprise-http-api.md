---
title: InfluxDB Anti-Entropy API
menu:
  enterprise_influxdb_1_7:
    menu: Enterprise HTTP API
    weight: 40
    parent: API
---

The following API endpoints are used by the [Anti-Entropy service](/enterprise_influxdb/v1.7/administration/anti-entropy) in InfluxDB Enterprise.

>**Note:** The Anti-Entropy API is only available when the Anti-Entropy service is enabled 
> in the data node configuration files. For information on the configuration settings, see 
> [Anti-Entropy settings](/enterprise_influxdb/v1.7/administration/config-data-nodes#anti-entropy-settings).

Base URL: `localhost:8086/shard-repair`

## General

### `/authorized`

### `/ping`

### `/role`


## Shards



### `/copy-shard-status`

### `/show-shards`

### `/status`

List shards that are in an inconsistent state and need repair.

#### ShardStatus

```Go
ShardStatus{
id                  string
database            string
retention_policy    string
start_time          string
end_time            string
expires             string
status              string
}
```

### `/repair`

Queue shard for repair of inconsistent state.

#### Parameters

* `id` ID of the shard to queue for repair

#### Responses

* `204` Successful operation
* `400` Bad request
* `500` Internal server error

### `/user`


### `/add-data`

### `/copy-shard`

### `/leave`

### `/remove-data`

### `/remove`

### `/restore-meta`

### `/update-data`

### `/truncate-shards`

### `/announce`

## Anti-Entropy

### `/cancel-repair/

#### Parameters

##### `id`


### `/status`


