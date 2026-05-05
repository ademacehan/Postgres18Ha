# PostgreSQL Logical Replication - Patroni Cluster to CDC Database
Bu proje, Patroni HA cluster’ınızdan harici bir CDC (Change Data Capture) veritabanına PostgreSQL logical replication kurulumunu sağlar.

***Mimari*** 

~~~

┌───────────────────────────────────────┐  
│   Patroni HA Cluster                  │  
│                                       │  
│   ┌──────┐  ┌──────┐  ┌──────┐         │  
│   │  14  │  │ 15   │  │ 16   │         │  
│   │etcd  │  │etcd  │  │etcd  │         │  
│   │pgb   │  │pgb   │  │pgb   │         │  
│   │pg    │  │pg    │  │pg    │         │  
│   └──────┘  └──────┘  └──────┘         │  
│                                       │  
│  Databases:                           │  
│  - personel (public.personel,         │  
│              public.personel_iletisim)│  
│  - odeme (public.personel_mass,       │  
│           public.personel_vergi,      │  
│           public.personel_yemek)      │  
└───────────────────────────────────────┘  
            │
            │ Logical Replication
            │ (pg_logical)
            ↓
┌─────────────────────────────────────┐
│   CDC Database Server               │
│                                     │
│  Database: cdc                      │
│  Schemas:                           │
│  - personel                         │
│    └─ personel                      │
│    └─ personel_iletisim             │
│  - odeme                            │
│    └─ personel_mass                 │
│    └─ personel_vergi                │
│    └─ personel_yemek                │
└─────────────────────────────────────┘
~~~
***Özellikler***
~~~
✅ Patroni HA cluster’dan bağımsız CDC veritabanına replication
✅ Her kaynak veritabanı için ayrı schema organizasyonu
✅ Kolay tablo ekleme/çıkarma
✅ Replication durumu monitoring
✅ Otomatik initial data sync (copy_data)
✅ Real-time change capture (INSERT, UPDATE, DELETE)
~~~
## Gereksinimler

### Patroni Cluster Tarafında  
***PostgreSQL Versiyonu:*** 10 veya üzeri (logical replication için)  
***Patroni Yapılandırması:*** Aşağıdaki parametreler ayarlanmalı
```bash 
patronictl edit-config <cluster-name> 
```
ile düzenleyin

postgresql:  
  parameters:  
    wal_level: logical  
    max_replication_slots: 10  
    max_wal_senders: 10  
    max_logical_replication_workers: 4  
    max_worker_processes: 8  

pg_hba.conf: CDC sunucusundan bağlantı izni   
***#Patroni'nin bootstrap.pg_hba kısmına ekleyin***    
host    personel        replication_user    <CDC_SERVER_IP>/32    md5  
host    odeme           replication_user    <CDC_SERVER_IP>/32    md5  
host    replication     replication_user    <CDC_SERVER_IP>/32    md5  

***Patroni Restart:*** Yapılandırma değişikliklerinden sonra   
```bash
patronictl -c /etc/patroni/patroni.yml restart <cluster-name>
```
### CDC Sunucu Tarafında
***PostgreSQL:*** 10 veya üzeri  
***Network:*** Patroni cluster’a erişim  
***Disk:*** Replicate edilecek veri boyutu + %20 buffer   

## Kurulum Adımları
***1. Yapılandırma Dosyasını Düzenleyin***
```bash 
nano replication_config.conf
```
Aşağıdaki bilgileri doldurun:  
```bash
SOURCE_HOST: Patroni cluster’ın master node IP’si (örn: “14” veya “192.168.1.14”)
SOURCE_PASSWORD: PostgreSQL postgres kullanıcı şifresi
TARGET_HOST: CDC sunucu IP’si
TARGET_PASSWORD: CDC sunucu postgres şifresi
REPL_PASSWORD: Oluşturulacak replication kullanıcısının şifresi
```

***2. Patroni yapılandırmasını düzenleyin***
```bash
patronictl -c /etc/patroni/patroni.yml edit-config <cluster-name>
```

***Yukarıdaki parametreleri ekleyin ve kaydedin***

### Cluster'ı restart edin
```bash
patronictl -c /etc/patroni/patroni.yml restart <cluster-name>
```
### Yapılandırmayı kontrol edin
```bash
psql -U postgres -c "SHOW wal_level;"  # 'logical' olmalı
```

***3. CDC Veritabanını Hazırlayın***
### CDC sunucusunda
```bash
psql -U postgres -f cdc_database_setup.sql
```
***4. Replication’ı Kurun***
```bash
chmod +x *.sh
./setup_logical_replication.sh
```

Script şunları yapacak:
Replication kullanıcısı oluşturur
Kaynak veritabanlarında publication’lar oluşturur
CDC veritabanında schema ve tabloları oluşturur
Subscription’ları kurar ve initial data sync başlatır

***5. Durumu Kontrol Edin***
```bash
./check_replication_status.sh
```

## Kullanım
### Yeni Tablo Ekleme
```bash
./add_table_to_replication.sh <kaynak_db> <kaynak_schema> <kaynak_tablo> <hedef_schema> [hedef_tablo]
```

Örnek:
#### personel veritabanından personel_maas tablosunu ekle
```bash
./add_table_to_replication.sh personel public personel_maas personel
```

#### odeme veritabanından yeni bir tablo ekle
```bash
./add_table_to_replication.sh odeme public personel_prim odeme
``` 

#### Tablo Çıkarma
```bash
./remove_table_from_replication.sh <kaynak_db> <kaynak_schema> <kaynak_tablo>
```

Örnek:
```bash
./remove_table_from_replication.sh personel public personel_maas
```
### Monitoring
#### Genel durum kontrolü
```bash
./check_replication_status.sh
```

## CDC veritabanında SQL ile monitoring
```bash
psql -U postgres -d cdc -c "SELECT * FROM replication_status;"
psql -U postgres -d cdc -c "SELECT * FROM table_statistics;"
psql -U postgres -d cdc -c "SELECT * FROM check_replication_health();"
```

## Sorun Giderme
### 1. Replication Çalışmıyor
-- Kaynak veritabanında
```bash
SELECT * FROM pg_replication_slots WHERE active = false;
```

-- CDC veritabanında
```bash
SELECT * FROM pg_stat_subscription;
```

### 2. Replication Lag Yüksek
-- Kaynak veritabanında lag kontrolü
```bash
SELECT slot_name, 
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) as lag
FROM pg_replication_slots;
```

Çözüm:
Network bağlantısını kontrol edin
CDC sunucu kaynaklarını artırın
max_logical_replication_workers parametresini artırın

### 3. Subscription Hata Veriyor
-- CDC veritabanında hataları görüntüle
```bash
SELECT * FROM pg_stat_subscription;
```

-- Subscription'ı yeniden başlat
```bash
ALTER SUBSCRIPTION personel_sub DISABLE;
ALTER SUBSCRIPTION personel_sub ENABLE;
```

### 4. Replication Slot Dolu
-- Kullanılmayan slot'ları temizle
```bash
SELECT pg_drop_replication_slot('slot_name');
```

### 5.Patroni Failover Sonrası
Patroni otomatik olarak yeni master’a geçiş yapar. Replication devam eder çünkü:

Replication slot’lar tüm node’larda senkronize edilir
Publication’lar cluster genelinde geçerlidir
Subscription’lar otomatik olarak yeni master’a bağlanır
Kontrol için:

### Yeni master'ı tespit edin
```bash
patronictl -c /etc/patroni/patroni.yml list
```

### Replication durumunu kontrol edin
```bash
./check_replication_status.sh
```

***Performans İpuçları***
***Replica Identity:*** FULL yerine DEFAULT kullanarak performansı artırabilirsiniz (primary key varsa)
```bash
ALTER TABLE tablename REPLICA IDENTITY DEFAULT;
```

***Batch Updates:*** Büyük toplu güncellemelerde replication lag’i izleyin  
***Vacuum:*** CDC veritabanında düzenli vacuum yapın  
### VACUUM ANALYZE;

Monitoring: Replication lag’i sürekli izleyin
```bash
watch -n 5 './check_replication_status.sh'
```

Güvenlik
***Şifre Güvenliği:*** replication_config.conf dosyasını koruyun
```bash
chmod 600 replication_config.conf
```

***Network:*** Sadece gerekli portları açın (5432)  
***SSL:*** Production’da SSL kullanın
```bash
CONNECTION 'host=... sslmode=require'
```

Yedekleme
CDC veritabanı sadece replica olduğu için:  
Kaynak veritabanlarını yedekleyin (Patroni’nin kendi backup stratejisi)
CDC veritabanı isteğe bağlı olarak yedeklenebilir (analiz için)  
Lisans  
Bu proje MIT lisansı altında lisanslanmıştır.  

### Destek
Sorularınız için:
GitHub Issues
PostgreSQL dokümantasyonu: https://www.postgresql.org/docs/current/logical-replication.html
Patroni dokümantasyonu: https://patroni.readthedocs.io/
Editor
