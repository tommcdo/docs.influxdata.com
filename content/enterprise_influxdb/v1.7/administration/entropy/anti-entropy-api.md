










## Anti-Entropy API 

### `/status`

#### GET

##### Description

List shards that are in an inconsistent state and in need of repair

##### Description

##### Parameters

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| local | query | Limit status check to local shards on the data node handling this request | No | boolean |

##### Responses

| Code | Description | Schema |
| ---- | ----------- | ------ |
| 200 | successful operation | object |

### `/repair`

#### POST

##### Description

Queue shard for repair of inconsistent state

##### Parameters

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| id | query | ID of shard to queue for repair | Yes | integer |

##### Responses

| Code | Description |
| ---- | ----------- |
| 204 | Successful operation |
| 400 | Bad request |
| 500 | Internal server error |

### `/cancel-repair`

#### POST

##### Description

Remove shard from repair queue on node(s)

##### Parameters

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| id | query | ID of shard to remove from repair queue | Yes | integer |
| local | query | Only remove shard from repair queue on node receiving the request | No | boolean |

##### Responses

| Code | Description |
| ---- | ----------- |
| 204 | successful operation |
| 400 | bad request |
| 500 | internal server error |

### Models

#### ShardStatus

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| id | string |  | No |
| database | string |  | No |
| retention_policy | string |  | No |
| start_time | string |  | No |
| end_time | string |  | No |
| expires | string |  | No |
| status | string |  | No |