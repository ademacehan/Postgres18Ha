### Mevcut alarm durumunun kontrol edilmesi
```bash
etcdctl --endpoints=http://127.0.0.1:2379 endpoint status --write-out=table
```

```bash
etcdctl --endpoints=http://127.0.0.1:2379 alarm list
```
Eğer çıktı "NOSPACE" alarmını gösteriyorsa

### Alarmları Temizleme
```bash 
etcdctl --endpoints=http://127.0.0.1:2379 alarm disarm
```