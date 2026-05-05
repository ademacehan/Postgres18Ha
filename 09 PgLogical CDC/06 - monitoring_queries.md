# monitoring_queries.sql
## PostgreSQL Logical Replication Monitoring SQL Sorguları

## ============================================
## KAYNAK VERİTABANI (Patroni Cluster) SORGULARI
## ============================================

## 1. Publication'ları listele
```bash
SELECT pubname, puballtables, pubinsert, pubupdate, pubdelete, pubtruncate
FROM pg_publication
ORDER BY pubname;
``` 

## 2. Publication'daki tabloları listele
```bash
SELECT pub.pubname, n.nspname, c.relname
FROM pg_publication pub
JOIN pg_publication_rel pr ON pub.oid = pr.prpubid
JOIN pg_class c ON pr.prrelid = c.oid
JOIN pg_namespace n ON c.relnamespace = n.oid
ORDER BY pub.pubname, n.nspname, c.relname;
```

## 3. Replication slot durumları
```bash
SELECT 
    slot_name,
    plugin,
    slot_type,
    database,
    active,
    active_pid,
    xmin,
    catalog_xmin,
    restart_lsn,
    confirmed_flush_lsn,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) as replication_lag,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), confirmed_flush_lsn)) as flush_lag
FROM pg_replication_slots
WHERE slot_type = 'logical'
ORDER BY slot_name;
```

## 4. WAL sender durumu
```bash
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    client_hostname,
    client_port,
    backend_start,
    state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    pg_size_pretty(pg_wal_lsn_diff(sent_lsn, replay_lsn)) as replay_lag,
    sync_state
FROM pg_stat_replication
ORDER BY application_name;
```

## 5. Aktif olmayan replication slot'ları bul
```bash
SELECT 
    slot_name,
    database,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) as lag,
    EXTRACT(EPOCH FROM (now() - last_active_time)) / 3600 as hours_inactive
FROM pg_replication_slots
WHERE slot_type = 'logical' 
AND active = false
ORDER BY hours_inactive DESC;
```

## 6. Replication slot'ların disk kullanımı
```bash
SELECT 
    slot_name,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) as retained_wal,
    pg_size_pretty(
        (pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) / 16777216)::bigint * 16777216
    ) as estimated_disk_usage
FROM pg_replication_slots
WHERE slot_type = 'logical'
ORDER BY pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) DESC;
```

## 7. Replica identity ayarlarını kontrol et
```bash
SELECT 
    n.nspname as schema_name,
    c.relname as table_name,
    CASE c.relreplident
        WHEN 'd' THEN 'DEFAULT'
        WHEN 'n' THEN 'NOTHING'
        WHEN 'f' THEN 'FULL'
        WHEN 'i' THEN 'INDEX'
    END as replica_identity
FROM pg_class c
JOIN pg_namespace n ON c.relnamespace = n.oid
WHERE c.relkind = 'r'
AND n.nspname IN ('public')
AND c.relname LIKE 'personel%'
ORDER BY n.nspname, c.relname;
```

## 8. WAL üretim hızı (son 5 dakika)
```bash
SELECT 
    pg_size_pretty(
        pg_wal_lsn_diff(pg_current_wal_lsn(), 
                       pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0')::text::pg_lsn)
    ) as wal_generated;
```

## ============================================
## CDC VERİTABANI SORGULARI
## ============================================

## 9. Subscription durumları
```bash
SELECT 
    subname,
    subenabled,
    subconninfo,
    subslotname,
    subpublications
FROM pg_subscription
ORDER BY subname;
```

## 10. Subscription istatistikleri
```bash
SELECT 
    subname,
    pid,
    received_lsn,
    last_msg_send_time,
    last_msg_receipt_time,
    latest_end_lsn,
    latest_end_time,
    CASE 
        WHEN last_msg_receipt_time IS NOT NULL THEN
            EXTRACT(EPOCH FROM (now() - last_msg_receipt_time))
        ELSE NULL
    END as lag_seconds
FROM pg_stat_subscription
ORDER BY subname;
```

## 11. Subscription worker durumu
```bash
SELECT 
    s.subname,
    w.worker_type,
    w.pid,
    w.leader_pid,
    w.relid::regclass as table_name,
    w.received_lsn,
    w.last_msg_send_time,
    w.last_msg_receipt_time,
    w.latest_end_lsn,
    w.latest_end_time
FROM pg_stat_subscription_workers w
JOIN pg_subscription s ON w.subid = s.oid
ORDER BY s.subname, w.worker_type;
```

## 12. Tablo istatistikleri (CDC veritabanı)
```bash
SELECT 
    schemaname,
    tablename,
    n_tup_ins as total_inserts,
    n_tup_upd as total_updates,
    n_tup_del as total_deletes,
    n_live_tup as live_rows,
    n_dead_tup as dead_rows,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_tuple_percent,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE schemaname IN ('personel', 'odeme')
ORDER BY schemaname, tablename;
```

## 13. Subscription conflict'leri
```bash
SELECT 
    s.subname,
    st.nspname as schema_name,
    st.relname as table_name,
    st.conflict_type,
    st.conflict_time,
    st.conflict_lsn
FROM pg_stat_subscription_stats st
JOIN pg_subscription s ON st.subid = s.oid
ORDER BY st.conflict_time DESC
LIMIT 100;
```

## 14. Tablo boyutları karşılaştırması
```bash
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
    pg_size_pretty(pg_indexes_size(schemaname||'.'||tablename)) as indexes_size,
    n_live_tup as row_count
FROM pg_stat_user_tables
WHERE schemaname IN ('personel', 'odeme')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## 15. Replication performans özeti
```bash
WITH subscription_stats AS (
    SELECT 
        subname,
        EXTRACT(EPOCH FROM (now() - last_msg_receipt_time)) as lag_seconds
    FROM pg_stat_subscription
)
SELECT 
    subname,
    CASE 
        WHEN lag_seconds IS NULL THEN 'UNKNOWN'
        WHEN lag_seconds < 1 THEN 'EXCELLENT'
        WHEN lag_seconds < 5 THEN 'GOOD'
        WHEN lag_seconds < 30 THEN 'FAIR'
        WHEN lag_seconds < 60 THEN 'POOR'
        ELSE 'CRITICAL'
    END as health_status,
    ROUND(lag_seconds::numeric, 2) as lag_seconds
FROM subscription_stats
ORDER BY lag_seconds DESC NULLS LAST;
```

## 16. Günlük replication aktivitesi
```bash
SELECT 
    schemaname,
    tablename,
    n_tup_ins as inserts_today,
    n_tup_upd as updates_today,
    n_tup_del as deletes_today,
    n_tup_ins + n_tup_upd + n_tup_del as total_changes
FROM pg_stat_user_tables
WHERE schemaname IN ('personel', 'odeme')
AND (n_tup_ins > 0 OR n_tup_upd > 0 OR n_tup_del > 0)
ORDER BY total_changes DESC;
```

## 17. Subscription bağlantı testi
```bash
SELECT 
    subname,
    subenabled,
    CASE 
        WHEN pid IS NOT NULL THEN 'CONNECTED'
        WHEN subenabled THEN 'ENABLED BUT NOT CONNECTED'
        ELSE 'DISABLED'
    END as connection_status
FROM pg_subscription s
LEFT JOIN pg_stat_subscription st ON s.oid = st.subid
ORDER BY subname;
```

## ============================================
## MAINTENANCE SORGULARI
## ============================================

## 18. Vacuum gerektiren tablolar (CDC)
```bash
SELECT 
    schemaname,
    tablename,
    n_dead_tup,
    n_live_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_tuple_percent,
    last_autovacuum
FROM pg_stat_user_tables
WHERE schemaname IN ('personel', 'odeme')
AND n_dead_tup > 1000
AND (n_dead_tup::float / NULLIF(n_live_tup + n_dead_tup, 0) > 0.1)
ORDER BY dead_tuple_percent DESC;
```

## 19. Index kullanım istatistikleri
```bash
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
WHERE schemaname IN ('personel', 'odeme')
ORDER BY idx_scan DESC;
```

## 20. Kullanılmayan index'ler
```bash
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
WHERE schemaname IN ('personel', 'odeme')
AND idx_scan = 0
AND indexrelname NOT LIKE '%_pkey'
ORDER BY pg_relation_size(indexrelid) DESC;
```