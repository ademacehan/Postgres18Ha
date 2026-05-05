setup_logical_replication.sh 
```bash
#!/bin/bash

# PostgreSQL Logical Replication Setup Script
# Bu script Patroni cluster'dan CDC veritabanına logical replication kurar

set -e

# Renkli çıktı için
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo -e "${GREEN}=== PostgreSQL Logical Replication Kurulum Scripti ===${NC}"

# Yapılandırma dosyasını yükle
if [ ! -f "replication_config.conf" ]; then
    echo -e "${RED}Hata: replication_config.conf dosyası bulunamadı!${NC}"
    exit 1
fi

source replication_config.conf

# Kaynak veritabanı bağlantı bilgileri (Patroni Cluster)
SOURCE_HOST=${SOURCE_HOST:-"14"}  # veya IP adresi
SOURCE_PORT=${SOURCE_PORT:-5432}
SOURCE_USER=${SOURCE_USER:-"postgres"}
SOURCE_PASSWORD=${SOURCE_PASSWORD}

# Hedef CDC veritabanı bağlantı bilgileri
TARGET_HOST=${TARGET_HOST}
TARGET_PORT=${TARGET_PORT:-5432}
TARGET_USER=${TARGET_USER:-"postgres"}
TARGET_PASSWORD=${TARGET_PASSWORD}
TARGET_DB="cdc"

# Replication user
REPL_USER=${REPL_USER:-"replication_user"}
REPL_PASSWORD=${REPL_PASSWORD}

echo -e "${YELLOW}1. Kaynak veritabanlarında replication kullanıcısı oluşturuluyor...${NC}"

# Her kaynak veritabanı için replication user oluştur
for db in "${SOURCE_DATABASES[@]}"; do
    echo -e "${GREEN}  -> $db veritabanında replication user oluşturuluyor...${NC}"
    PGPASSWORD=$SOURCE_PASSWORD psql -h $SOURCE_HOST -p $SOURCE_PORT -U $SOURCE_USER -d $db <<EOF
-- Replication kullanıcısı oluştur (zaten varsa hata vermez)
DO \$\$
BEGIN
    IF NOT EXISTS (SELECT FROM pg_catalog.pg_roles WHERE rolname = '$REPL_USER') THEN
        CREATE ROLE $REPL_USER WITH REPLICATION LOGIN PASSWORD '$REPL_PASSWORD';
    END IF;
END
\$\$;

-- Gerekli yetkileri ver
GRANT CONNECT ON DATABASE $db TO $REPL_USER;
GRANT USAGE ON SCHEMA public TO $REPL_USER;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO $REPL_USER;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO $REPL_USER;
EOF
done

echo -e "${YELLOW}2. Kaynak veritabanlarında publication oluşturuluyor...${NC}"

# personel veritabanı için publication
PGPASSWORD=$SOURCE_PASSWORD psql -h $SOURCE_HOST -p $SOURCE_PORT -U $SOURCE_USER -d personel <<EOF
-- Publication oluştur (zaten varsa önce sil)
DROP PUBLICATION IF EXISTS personel_pub;
CREATE PUBLICATION personel_pub FOR TABLE public.personel, public.personel_iletisim;

-- Replica identity ayarla (UPDATE ve DELETE için gerekli)
ALTER TABLE public.personel REPLICA IDENTITY FULL;
ALTER TABLE public.personel_iletisim REPLICA IDENTITY FULL;
EOF

echo -e "${GREEN}  -> personel_pub publication oluşturuldu${NC}"

# odeme veritabanı için publication
PGPASSWORD=$SOURCE_PASSWORD psql -h $SOURCE_HOST -p $SOURCE_PORT -U $SOURCE_USER -d odeme <<EOF
-- Publication oluştur (zaten varsa önce sil)
DROP PUBLICATION IF EXISTS odeme_pub;
CREATE PUBLICATION odeme_pub FOR TABLE public.personel_mass, public.personel_vergi, public.personel_yemek;

-- Replica identity ayarla
ALTER TABLE public.personel_mass REPLICA IDENTITY FULL;
ALTER TABLE public.personel_vergi REPLICA IDENTITY FULL;
ALTER TABLE public.personel_yemek REPLICA IDENTITY FULL;
EOF

echo -e "${GREEN}  -> odeme_pub publication oluşturuldu${NC}"

echo -e "${YELLOW}3. CDC veritabanında schema ve tablolar oluşturuluyor...${NC}"

PGPASSWORD=$TARGET_PASSWORD psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d $TARGET_DB <<EOF
-- Schema'ları oluştur
CREATE SCHEMA IF NOT EXISTS personel;
CREATE SCHEMA IF NOT EXISTS odeme;

-- personel schema tabloları
DROP TABLE IF EXISTS personel.personel CASCADE;
CREATE TABLE personel.personel (LIKE personel.personel INCLUDING ALL);

DROP TABLE IF EXISTS personel.personel_iletisim CASCADE;
CREATE TABLE personel.personel_iletisim (LIKE personel.personel_iletisim INCLUDING ALL);

-- odeme schema tabloları
DROP TABLE IF EXISTS odeme.personel_mass CASCADE;
CREATE TABLE odeme.personel_mass (LIKE odeme.personel_mass INCLUDING ALL);

DROP TABLE IF EXISTS odeme.personel_vergi CASCADE;
CREATE TABLE odeme.personel_vergi (LIKE odeme.personel_vergi INCLUDING ALL);

DROP TABLE IF EXISTS odeme.personel_yemek CASCADE;
CREATE TABLE odeme.personel_yemek (LIKE odeme.personel_yemek INCLUDING ALL);
EOF

echo -e "${GREEN}  -> CDC veritabanında schema ve tablolar oluşturuldu${NC}"

echo -e "${YELLOW}4. Subscription'lar oluşturuluyor...${NC}"

# personel veritabanı subscription
PGPASSWORD=$TARGET_PASSWORD psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d $TARGET_DB <<EOF
-- Mevcut subscription'ı sil (varsa)
DROP SUBSCRIPTION IF EXISTS personel_sub;

-- Yeni subscription oluştur
CREATE SUBSCRIPTION personel_sub
    CONNECTION 'host=$SOURCE_HOST port=$SOURCE_PORT dbname=personel user=$REPL_USER password=$REPL_PASSWORD'
    PUBLICATION personel_pub
    WITH (
        copy_data = true,
        create_slot = true,
        enabled = true,
        slot_name = 'personel_slot'
    );
EOF

echo -e "${GREEN}  -> personel_sub subscription oluşturuldu${NC}"

# odeme veritabanı subscription
PGPASSWORD=$TARGET_PASSWORD psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d $TARGET_DB <<EOF
-- Mevcut subscription'ı sil (varsa)
DROP SUBSCRIPTION IF EXISTS odeme_sub;

-- Yeni subscription oluştur
CREATE SUBSCRIPTION odeme_sub
    CONNECTION 'host=$SOURCE_HOST port=$SOURCE_PORT dbname=odeme user=$REPL_USER password=$REPL_PASSWORD'
    PUBLICATION odeme_pub
    WITH (
        copy_data = true,
        create_slot = true,
        enabled = true,
        slot_name = 'odeme_slot'
    );
EOF

echo -e "${GREEN}  -> odeme_sub subscription oluşturuldu${NC}"

echo -e "${GREEN}=== Kurulum tamamlandı! ===${NC}"
echo -e "${YELLOW}Replication durumunu kontrol etmek için check_replication_status.sh scriptini çalıştırın.${NC}"
````