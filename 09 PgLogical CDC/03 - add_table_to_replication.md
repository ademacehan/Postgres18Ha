# add_table_to_replication.sh 

```bash
#!/bin/bash
# Mevcut replication'a yeni tablo ekleyen script

set -e
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

if [ ! -f "replication_config.conf" ]; then
    echo -e "${RED}Hata: replication_config.conf dosyası bulunamadı!${NC}"
    exit 1
fi
source replication_config.conf

# Parametreleri al
if [ $# -lt 4 ]; then
    echo -e "${RED}Kullanım: $0 <kaynak_db> <kaynak_schema> <kaynak_tablo> <hedef_schema>${NC}"
    echo -e "${YELLOW}Örnek: $0 personel public personel_maas personel${NC}"
    exit 1
fi

SOURCE_DB=$1
SOURCE_SCHEMA=$2
SOURCE_TABLE=$3
TARGET_SCHEMA=$4
TARGET_TABLE=${5:-$SOURCE_TABLE}  # Hedef tablo adı belirtilmezse kaynak ile aynı

echo -e "${GREEN}=== Yeni Tablo Replication'a Ekleniyor ===${NC}"
echo -e "Kaynak: ${YELLOW}$SOURCE_DB.$SOURCE_SCHEMA.$SOURCE_TABLE${NC}"
echo -e "Hedef: ${YELLOW}cdc.$TARGET_SCHEMA.$TARGET_TABLE${NC}\n"

# Publication adını belirle
if [ "$SOURCE_DB" == "personel" ]; then
    PUB_NAME="personel_pub"
    SUB_NAME="personel_sub"
elif [ "$SOURCE_DB" == "odeme" ]; then
    PUB_NAME="odeme_pub"
    SUB_NAME="odeme_sub"
else
    echo -e "${RED}Hata: Desteklenmeyen kaynak veritabanı: $SOURCE_DB${NC}"
    echo -e "${YELLOW}Desteklenen veritabanları: personel, odeme${NC}"
    exit 1
fi

echo -e "${YELLOW}1. Kaynak tabloya replica identity ekleniyor...${NC}"
PGPASSWORD=$SOURCE_PASSWORD psql -h $SOURCE_HOST -p $SOURCE_PORT -U $SOURCE_USER -d $SOURCE_DB <<EOF
ALTER TABLE $SOURCE_SCHEMA.$SOURCE_TABLE REPLICA IDENTITY FULL;
EOF

echo -e "${YELLOW}2. Tablo publication'a ekleniyor...${NC}"
PGPASSWORD=$SOURCE_PASSWORD psql -h $SOURCE_HOST -p $SOURCE_PORT -U $SOURCE_USER -d $SOURCE_DB <<EOF
ALTER PUBLICATION $PUB_NAME ADD TABLE $SOURCE_SCHEMA.$SOURCE_TABLE;
EOF

echo -e "${YELLOW}3. CDC veritabanında hedef tablo oluşturuluyor...${NC}"
PGPASSWORD=$TARGET_PASSWORD psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d cdc <<EOF
-- Schema oluştur (yoksa)
CREATE SCHEMA IF NOT EXISTS $TARGET_SCHEMA;

-- Tabloyu kopyala
DROP TABLE IF EXISTS $TARGET_SCHEMA.$TARGET_TABLE CASCADE;
CREATE TABLE $TARGET_SCHEMA.$TARGET_TABLE (LIKE $SOURCE_SCHEMA.$SOURCE_TABLE INCLUDING ALL);
EOF

echo -e "${YELLOW}4. Subscription yenileniyor...${NC}"
PGPASSWORD=$TARGET_PASSWORD psql -h $TARGET_HOST -p $TARGET_PORT -U $TARGET_USER -d cdc <<EOF
-- Subscription'ı yenile
ALTER SUBSCRIPTION $SUB_NAME REFRESH PUBLICATION;
EOF

echo -e "${GREEN}=== Tablo başarıyla eklendi! ===${NC}"
echo -e "${YELLOW}Durumu kontrol etmek için: ./check_replication_status.sh${NC}"
```