# PostgreSQL Logical Replication Yapılandırma Dosyası
# Bu dosyayı kendi ortamınıza göre düzenleyin

# Kaynak Patroni Cluster Bilgileri
SOURCE_HOST="14"  # veya tam IP adresi: 192.168.1.14
SOURCE_PORT="5432"
SOURCE_USER="postgres"
SOURCE_PASSWORD="your_source_password"

# Kaynak veritabanları
SOURCE_DATABASES=("personel" "odeme")

# Hedef CDC Sunucu Bilgileri
TARGET_HOST="your_cdc_server_ip"
TARGET_PORT="5432"
TARGET_USER="postgres"
TARGET_PASSWORD="your_target_password"

# Replication Kullanıcı Bilgileri
REPL_USER="replication_user"
REPL_PASSWORD="your_replication_password"