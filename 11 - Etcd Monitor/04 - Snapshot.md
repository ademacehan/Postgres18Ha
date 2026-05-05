### Snapshot yedekleme işlemi

#### Snapshot alma
```bash
 etcdctl snapshot save /data/etcd/etcd-snapshot-$(date +%Y%m%d-%H%M%S).db
 ```

#### Snapshot durumunu kontrol et
```bash
etcdutl snapshot status etcd-snapshot-20251211-235714.db -w table
```
|HASH|REVISION|TOTAL KEYS | TOTAL SIZE | VERSION |
|--|--|--|------------|---------|
|542de692|11862|10012|1.4 MB|3.6.0|

#### SNAPSHOT restore