#### 1. Replication gecikmiş / durmuş
```bash
SELECT usename, state, sync_state, sent_lsn, write_lsn, flush_lsn, replay_lsn
FROM pg_stat_replication;
```

#### 2. Archivng (pgBackRest, WAL-G, wal-e …) çalışmıyor
```bash 
SELECT * FROM pg_stat_archiver;
```

#### 3. çalışmyan replication slot
```bash
SELECT slot_name, plugin, slot_type, database, active, restart_lsn
FROM pg_replication_slots;
```

```bash
SELECT pg_drop_replication_slot('slot_name');
```

```bash
CHECKPOINT;
```