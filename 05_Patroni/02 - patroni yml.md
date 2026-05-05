```bash
vi /etc/patroni/patroni.yml
```


```bash
scope: postgres-cluster
namespace: takbis
name: node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: 172.20.38.14:8008
  allowlist_include_members: true
  allowlist:
    - 172.20.38.14

etcd3:
  hosts: 172.20.38.14:2379,172.20.38.15:2379,172.20.38.16:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 5
    max_timelines_history: 20
    maximum_lag_on_failover: 10485760
    master_start_timeout: 300
    synchronous_mode: True
    synchronous_mode_strict: True
    synchronous_node_count: 1
    failsafe_mode: true
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:
        max_connections: 1000
        max_replication_slots: 10
        shared_buffers: 2048MB
        temp_buffers: 8MB
        effective_cache_size: 5120MB
        work_mem: 16MB
        max_files_per_process: 1000
        tcp_keepalives_idle: 60
        tcp_keepalives_interval: 3
        tcp_keepalives_count: 3
        wal_level: logical
        min_wal_size: 2GB
        max_wal_size: 4GB
        wal_buffers: 16MB
        wal_keep_size: 1GB
        wal_compression: lz4
        wal_log_hints: on
        max_wal_senders: 10
        max_parallel_workers_per_gather: 2
        parallel_setup_cost: 10000
        archive_mode: on
        archive_command: 'wal-g wal-push --config /etc/wal-g/wal-g.json %p'
        archive_timeout: 600
        hot_standby: on
        shared_preload_libraries: pg_stat_statements
        pg_stat_statements.max: 10000
        pg_stat_statements.track: all
        track_io_timing: on
        dynamic_shared_memory_type: posix
        log_connections: True
        log_destination: stderr
        logging_collector: on
        log_directory: /Log
        log_filename: 'postgresql-%d.log'
        log_truncate_on_rotation: on
        log_rotation_age: 1d
        log_rotation_size: 0
        log_line_prefix: '%a->%u->%d->%m [%p] i '
        log_min_duration_statement: 1000
        log_parameter_max_length: 1000
        log_autovacuum_min_duration: 3000
        log_statement: ddl
        log_timezone: 'Europe/Istanbul'
        datestyle: iso, mdy
        timezone: 'Europe/Istanbul'
        lc_messages: en_US.UTF-8
        lc_monetary: en_US.UTF-8
        lc_numeric: en_US.UTF-8
        lc_time: en_US.UTF-8
        default_text_search_config: pg_catalog.english
        temp_file_limit: 4.0GB
        idle_in_transaction_session_timeout: 900000
        random_page_cost: 1
        effective_io_concurrency: 200
        autovacuum_max_workers: 2
        autovacuum_naptime: 15
        autovacuum_vacuum_cost_delay: 2
        autovacuum_vacuum_cost_limit: 200
        vacuum_cost_page_miss: 2
        vacuum_cost_limit: 200
        autovacuum_vacuum_scale_factor: 0.05
        autovacuum_analyze_scale_factor: 0.01
        bgwriter_delay: 100
        bgwriter_lru_maxpages: 400
        bgwriter_lru_multiplier: 4
        vacuum_cost_page_hit: 1
        vacuum_cost_page_dirty: 20
        checkpoint_timeout: 10min
        enable_mergejoin: on
        maintenance_work_mem: 256MB
        password_encryption: scram-sha-256
        wal_receiver_timeout: 30s
        wal_sender_timeout: 30s
        autovacuum_vacuum_insert_scale_factor: 0.1
        autovacuum_vacuum_insert_threshold: 1000
        max_locks_per_transaction: 64
        escape_string_warning: True
        standard_conforming_strings: True
        unix_socket_directories: /var/run/postgresql, /tmp

        pg_hba:
        - local all all          peer
        - host    postgres        postgres        172.20.38.14/32         trust
        - host    postgres        postgres        172.20.38.15/32         trust
        - host    postgres        postgres        172.20.38.16/32         trust
        - host    bouncer         bouncer         172.20.38.14/32         scram-sha-256
        - host    bouncer         bouncer         172.20.38.15/32         scram-sha-256
        - host    bouncer         bouncer         172.20.38.16/32         scram-sha-256
        - host    aacehan         aacehan         172.20.38.14/32         scram-sha-256
        - host    aacehan         aacehan         172.20.38.15/32         scram-sha-256
        - host    aacehan         aacehan         172.20.38.16/32         scram-sha-256
        - host    bouncer         bouncer         127.0.0.1/32            trust
        - host    replication     replicator      172.20.38.14/32         trust
        - host    replication     replicator      172.20.38.15/32         trust
        - host    replication     replicator      172.20.38.16/32         trust


    recovery_conf:
      restore_command: cp /home/postgres/archived/%f %p
  initdb:
    - encoding: UTF8
    - data-checksums
    - waldir: /Wallog/pg_wal
postgresql:
  cluster_name: takbis
  listen: 172.20.38.14:5432
  connect_address: 172.20.38.14:5432
  use_unix_socket: true
  data_dir: /data/pg18/
  bin_dir: /usr/pgsql-18/bin
  config_dir: /data/pg18
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: takbis2025
    superuser:
      username: postgres
      password: takbis2025
  create_replica_methods:
    - basebackup
  basebackup:
    checkpoint: fast

tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
log:
  level: INFO
  dir: /var/log/patroni
```
*** ttl:*** değeri retry_timeout + loop_wait den büyük olmalıdır.
***loop_wait: *** 
    retry_timeout: 10
    max_timelines_history: 20
    maximum_lag_on_failover: 10485760
    master_start_timeout: 300
    synchronous_mode: True
    synchrono
***scope :***  patroni cluster name dir.  


***log  :***  Patroni log yönetimi burada yapılmaktadır.  
&nbsp;&nbsp;&nbsp;&nbsp;*****level :***** default olarak INFO yazılır.  
&nbsp;&nbsp;&nbsp;&nbsp;**LOGLAMA** seviyeleri EXCEPTION, LOG, CRITICAL, ERROR, WARNING, INFO, DEBUG, NOTSET




