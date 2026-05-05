#remove_table_from_replication.sh 
- Patroni Cluster için PostgreSQL Logical Replication Yapılandırması
- Bu SQL dosyası Patroni cluster'ın her bir node'unda çalıştırılmalıdır

## 1. postgresql.conf parametrelerini kontrol et
- Patroni üzerinden bu parametreleri ayarlamak için:
```bash 
patronictl edit-config <cluster-name>
```

/*
Patroni YAML yapılandırmasına eklenecek parametreler:

postgresql:
  parameters:
    wal_level: logical
    max_replication_slots: 10
    max_wal_senders: 10
    max_logical_replication_workers: 4
    max_worker_processes: 8
    shared_preload_libraries: 'pg_stat_statements'
*/

-- 2. pg_hba.conf yapılandırması
-- Patroni'nin pg_hba.conf dosyasına şu satırları ekleyin:

/*
# Logical replication için
host    personel        replication_user    <CDC_SERVER_IP>/32    md5
host    odeme           replication_user    <CDC_SERVER_IP>/32    md5
host    replication     replication_user    <CDC_SERVER_IP>/32    md5
*/

-- 3. Mevcut replication slot'ları kontrol et
SELECT slot_name, plugin, slot_type, database, active, 
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) as lag
FROM pg_replication_slots;

-- 4. Aktif publication'ları kontrol et
SELECT pubname, puballtables, pubinsert, pubupdate, pubdelete
FROM pg_publication;

-- 5. WAL seviyesini kontrol et
SHOW wal_level;  -- 'logical' olmalı

-- 6. Replication slot limitleri kontrol et
SHOW max_replication_slots;  -- En az 10 olmalı
SHOW max_wal_senders;        -- En az 10 olmalı

-- 7. Kullanılmayan replication slot'ları temizle (gerekirse)
-- SELECT pg_drop_replication_slot('slot_name');

-- 8. Publication'daki tabloları görüntüle
SELECT schemaname, tablename 
FROM pg_publication_tables 
WHERE pubname IN ('personel_pub', 'odeme_pub');

-- 9. Replication lag'i izle
SELECT 
    slot_name,
    database,
    active,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) as replication_lag,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), confirmed_flush_lsn)) as flush_lag
FROM pg_replication_slots
WHERE slot_type = 'logical';

-- 10. Patroni cluster durumunu kontrol et (bash'ten)
-- patronictl -c /etc/patroni/patroni.yml list