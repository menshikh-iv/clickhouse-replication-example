# Play with cluster

### Cleanup old data

If you're Tim, you need to do these first, because you tried to run previous version

```bash
sudo rm -rf data/*
```

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
    userid UInt32,
    timestamp DateTime DEFAULT now()
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{cluster}/{shard}/default/test', '{replica}')
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp, userid)
```

### Insert smth into every replica

```bash
docker exec -ti clickhouse-replication-example_ch-client_1 /bin/bash -c 'clickhouse-client --host ch1 --query "INSERT INTO testdb.test(userid) VALUES (1)(2)(3)(4)(5)"'
docker exec -ti clickhouse-replication-example_ch-client_1 /bin/bash -c 'clickhouse-client --host ch2 --query "INSERT INTO testdb.test(userid) VALUES (1)(2)(3)(4)(5)"'
docker exec -ti clickhouse-replication-example_ch-client_1 /bin/bash -c 'clickhouse-client --host ch3 --query "INSERT INTO testdb.test(userid) VALUES (1)(2)(3)(4)(5)"'
```

### Check that we have same # of rows on each replica

```bash
docker exec -ti clickhouse-replication-example_ch-client_1 /bin/bash -c 'clickhouse-client --host ch1 --query "SELECT COUNT(*) FROM testdb.test"'
docker exec -ti clickhouse-replication-example_ch-client_1 /bin/bash -c 'clickhouse-client --host ch1 --query "SELECT COUNT(*) FROM testdb.test"'
docker exec -ti clickhouse-replication-example_ch-client_1 /bin/bash -c 'clickhouse-client --host ch1 --query "SELECT COUNT(*) FROM testdb.test"'
```
