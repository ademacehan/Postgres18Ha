# cdc_database_setup.sql

## 1. CDC veritabanını oluştur (yoksa)
```bash
CREATE DATABASE cdc
    WITH 
    OWNER = postgres
    ENCODING = 'UTF8'
    LC_COLLATE = 'tr_TR.UTF-8'
    LC_CTYPE = 'tr_TR.UTF-8'
    TABLESPACE = pg_default
    CONNECTION LIMIT = -1;
```

```bash
\c cdc
```

## 2. Schema'ları oluştur
```bash
CREATE SCHEMA IF NOT EXISTS personel;
CREATE SCHEMA IF NOT EXISTS odeme;
```

## 3. Extension'ları yükle (gerekirse)
```bash
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements";
```

## 4. Replication monitoring view'ı oluştur
```bash
CREATE OR REPLACE VIEW replication_status AS
SELECT 
    s.subname as subscription_name,
    s.subenabled as enabled,
    st.received_lsn,
    st.last_msg_send_time,
    st.last_msg_receipt_time,
    st.latest_end_lsn,
    st.latest_end_time,
    CASE 
        WHEN st.last_msg_receipt_time IS NOT NULL THEN
            EXTRACT(EPOCH FROM (now() - st.last_msg_receipt_time))
        ELSE NULL
    END as lag_seconds
FROM pg_subscription s
LEFT JOIN pg_stat_subscription st ON s.oid = st.subid;
```

## 5. Tablo istatistikleri view'ı
```bash
CREATE OR REPLACE VIEW table_statistics AS
SELECT 
    schemaname,
    tablename,
    n_tup_ins as inserts,
    n_tup_upd as updates,
    n_tup_del as deletes,
    n_live_tup as live_rows,
    n_dead_tup as dead_rows,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE schemaname IN ('personel', 'odeme')
ORDER BY schemaname, tablename;
```

## 6. Replication conflict'leri izlemek için view
```bash
CREATE OR REPLACE VIEW replication_conflicts AS
SELECT 
    subname,
    nspname,
    relname,
    conflict_type,
    conflict_time
FROM pg_stat_subscription_stats
ORDER BY conflict_time DESC;
```

## 7. Monitoring fonksiyonu
```bash
CREATE OR REPLACE FUNCTION check_replication_health()
RETURNS TABLE (
    check_name text,
    status text,
    details text
) AS $$
BEGIN
    ## Subscription durumu kontrolü
    RETURN QUERY
    SELECT 
        'Subscription Status'::text,
        CASE WHEN subenabled THEN 'OK' ELSE 'WARNING' END::text,
        subname::text
    FROM pg_subscription;
    
    ## Replication lag kontrolü
    RETURN QUERY
    SELECT 
        'Replication Lag'::text,
        CASE 
            WHEN lag_seconds IS NULL THEN 'UNKNOWN'
            WHEN lag_seconds < 60 THEN 'OK'
            WHEN lag_seconds < 300 THEN 'WARNING'
            ELSE 'CRITICAL'
        END::text,
        COALESCE(lag_seconds::text || ' seconds', 'N/A')::text
    FROM replication_status;
    
    ## Dead tuple kontrolü
    RETURN QUERY
    SELECT 
        'Dead Tuples'::text,
        CASE 
            WHEN n_dead_tup > n_live_tup * 0.2 THEN 'WARNING'
            ELSE 'OK'
        END::text,
        schemaname || '.' || tablename || ': ' || n_dead_tup::text
    FROM pg_stat_user_tables
    WHERE schemaname IN ('personel', 'odeme')
    AND n_dead_tup > 1000;
END;
$$ LANGUAGE plpgsql;
```

## 8. Yetkilendirme
```bash
GRANT USAGE ON SCHEMA personel TO postgres;
GRANT USAGE ON SCHEMA odeme TO postgres;
GRANT SELECT ON ALL TABLES IN SCHEMA personel TO postgres;
GRANT SELECT ON ALL TABLES IN SCHEMA odeme TO postgres;
GRANT SELECT ON replication_status TO postgres;
GRANT SELECT ON table_statistics TO postgres;
GRANT EXECUTE ON FUNCTION check_replication_health() TO postgres;
```

## 9. Otomatik vacuum ayarları
```bash
ALTER DATABASE cdc SET autovacuum = on;
ALTER DATABASE cdc SET autovacuum_vacuum_scale_factor = 0.1;
ALTER DATABASE cdc SET autovacuum_analyze_scale_factor = 0.05;
```

## Kullanım örnekleri:
```bash
SELECT * FROM replication_status;
SELECT * FROM table_statistics;
SELECT * FROM check_replication_health();
```