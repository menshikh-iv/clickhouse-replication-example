# Play with cluster

### Up cluster

```bash
docker-compose up
```

### Check that clickhouse are up

```bash
docker exec -ti clickhouse-replication-example_ch-client_1 /bin/bash
clickhouse-client --host ch1
select 1;
```

### Create dabase on cluster

```sql
CREATE DATABASE IF NOT EXISTS testdb ON CLUSTER '{cluster}';
```


### Create distributed table on cluster

```sql
CREATE TABLE IF NOT EXISTS testdb.test ON CLUSTER '{cluster}'
(
    timestamp DateTime,
    contractid UInt32,
    userid UInt32
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{cluster}/{shard}/default/test', '{replica}')
PARTITION BY toYYYYMM(timestamp)
ORDER BY (contractid, toDate(timestamp), userid)
SAMPLE BY userid;
```
