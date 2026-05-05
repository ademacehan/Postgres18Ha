# check_replication_status.sh

```bash
#!/bin/bash

# Replication durumunu kontrol eden script

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

if [ ! -f "replication_config.conf" ]; then
    echo -e "${RED}Hata: replication_config.conf dosyası bulunamadı!${NC}"
    exit 1
fi

source replication_config.conf

echo -e "${BLUE}=== Replication Durum Kontrolü ===${NC}\n"

echo -e "${YELLOW}1. Kaynak veritabanlarındaki publication durumu:${NC}"
for db in "${SOURCE_DATABASES[@]}"; do
    echo -e "${GREEN}  -> $db veritabanı:${NC}"
    PGPASSWORD=$SOURCE_PASSWORD psql -h $SOURCE_HOST -p $SOURCE_PORT -U $SOURCE_USER -d $db -c "SELECT * FROM pg_publication;"
    echo ""
done

echo -e "${YELLOW}2. Kaynak veritabanlarındaki replication slot durumu:${NC}"
PGPASSWORD=$SOURCE_PASSWORD psql -h $SOURCE_HOST -p $SOURCE_PORT -U $SOURCE_USER -d postgres -c "
SELECT slot_name, plugin, slot_type, database, active, 
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) as replication_lag
FROM pg_replication_slots;
"

echo -e "${YELLOW}3. CDC veritabanındaki subscription durumu:${NC}"
PGPASSWORD=$TARGET_PASSWORD psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d cdc -c "
SELECT subname, subenabled, 
       CASE 
           WHEN subenabled THEN 'Aktif'
           ELSE 'Pasif'
       END as durum
FROM pg_subscription;
"

echo -e "${YELLOW}4. Subscription istatistikleri:${NC}"
PGPASSWORD=$TARGET_PASSWORD psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d cdc -c "
SELECT subname, 
       received_lsn,
       last_msg_send_time,
       last_msg_receipt_time,
       latest_end_lsn,
       latest_end_time
FROM pg_stat_subscription;
"

echo -e "${YELLOW}5. CDC veritabanındaki tablo satır sayıları:${NC}"
PGPASSWORD=$TARGET_PASSWORD psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d cdc <<EOF
SELECT 
    schemaname,
    tablename,
    (SELECT COUNT(*) FROM personel.personel) as row_count
FROM pg_tables 
WHERE schemaname = 'personel' AND tablename = 'personel'
UNION ALL
SELECT 
    schemaname,
    tablename,
    (SELECT COUNT(*) FROM personel.personel_iletisim) as row_count
FROM pg_tables 
WHERE schemaname = 'personel' AND tablename = 'personel_iletisim'
UNION ALL
SELECT 
    schemaname,
    tablename,
    (SELECT COUNT(*) FROM odeme.personel_mass) as row_count
FROM pg_tables 
WHERE schemaname = 'odeme' AND tablename = 'personel_mass'
UNION ALL
SELECT 
    schemaname,
    tablename,
    (SELECT COUNT(*) FROM odeme.personel_vergi) as row_count
FROM pg_tables 
WHERE schemaname = 'odeme' AND tablename = 'personel_vergi'
UNION ALL
SELECT 
    schemaname,
    tablename,
    (SELECT COUNT(*) FROM odeme.personel_yemek) as row_count
FROM pg_tables 
WHERE schemaname = 'odeme' AND tablename = 'personel_yemek';
EOF

echo -e "\n${GREEN}=== Kontrol tamamlandı ===${NC}"
```