### En son revision numarasının alınması
```bash 
rev=$(etcdctl --endpoints=http://127.0.0.1:2379 endpoint status --write-out=json | egrep -o '"revision":[0-9]*' | cut -d: -f2)
```

### revision'a kadar olan eski verileri temizlenmesi
```bash 
etcdctl --endpoints=http://127.0.0.1:2379 compact $rev
```

### etcd veri dosyasının defrag edilmesi
```bash
etcdctl defrag
```

### Kontrol
```bash 
etcdctl --endpoints=http://127.0.0.1:2379 endpoint status --write-out=table
```

### Crontab
```bash 
/usr/local/bin/etcd_cleanup.sh
```

```bash 
#!/bin/bash

# Değişkenler
export ETCDCTL_API=3
ENDPOINTS="http://127.0.0.1:2379"

# 1. En son revision numarasını al
REV=$(etcdctl --endpoints=$ENDPOINTS endpoint status --write-out=json | grep -o '"revision":[0-9]*' | cut -d: -f2)

# 2. Eski revision'ları temizle (Compaction)
echo "Compacting up to revision $REV..."
etcdctl --endpoints=$ENDPOINTS compact $REV

# 3. Diskteki boş alanı geri kazan (Defragmentation)
# NOT: Bu işlem sırasında etcd milisaniyelik duraksayabilir.
echo "Defragmenting etcd storage..."
etcdctl --endpoints=$ENDPOINTS defrag

# 4. Eğer varsa alarmları temizle
echo "Disarming any active alarms..."
etcdctl --endpoints=$ENDPOINTS alarm disarm

echo "Etcd cleanup tamamlandı at $(date)"
```

```bash 
chmod +x /usr/local/bin/etcd_cleanup.sh
```

```bash
crontab -e
```

```bash
00 01 * * * /usr/local/bin/etcd_cleanup.sh >> /var/log/etcd_maintenance.log 2>&1
```