### lag izleme
Aşağıdaki Sql komutu ile lag izlem işlemi gerçekleştirilir.
```bash
SELECT
    application_name                                    AS node_name,
    client_addr                                         AS node_ip,
    state                                               AS replication_state,
    (pg_current_wal_lsn() - sent_lsn)                   AS network_lag_bytes,
    (sent_lsn - write_lsn)                              AS write_lag_bytes,
    (write_lsn - flush_lsn)                             AS flush_lag_bytes,
    (flush_lsn - replay_lsn)                            AS replay_lag_bytes,
    (pg_current_wal_lsn() - replay_lsn)                 AS total_lag_bytes,
    pg_size_pretty(pg_current_wal_lsn() - replay_lsn)   AS readable_total_lag
FROM pg_stat_replication;
```
*** watch ile izlemek için ***
10 sn bir lag durumu hakkında bilgi verir.
```bash

watch -n 10 "psql -h db01 -p 6432 -U aacehan -d aacehan -c \
\"SELECT
    application_name                                    AS node_name,
    client_addr                                         AS node_ip,
    state                                               AS replication_state,
    (pg_current_wal_lsn() - sent_lsn)                   AS network_lag_bytes,
    (sent_lsn - write_lsn)                              AS write_lag_bytes,
    (write_lsn - flush_lsn)                             AS flush_lag_bytes,
    (flush_lsn - replay_lsn)                            AS replay_lag_bytes,
    (pg_current_wal_lsn() - replay_lsn)                 AS total_lag_bytes,
    pg_size_pretty(pg_current_wal_lsn() - replay_lsn)   AS readable_total_lag
FROM pg_stat_replication;\""
```

Geçikmş veya geride kalmış repliikasyon  
kayıtlarının tespiti
```bash

psql -h db01 -p 6432 -U aacehan -d aacehan -c \
"SELECT client_addr, application_name, usename, state, sync_state, sent_lsn, write_lsn, flush_lsn, replay_lsn
FROM pg_stat_replication;"
```