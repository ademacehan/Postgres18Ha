### Get leader 
Lider bilgisini almak için  
```bash
 etcdctl get /takbis/postgres-cluster/leader
```


### Durum bilgisini almak için 
```bash
 etcdctl get /takbis/postgres-cluster/status
```

### Sync Sunucu bilgisini almak için 
```bash
etcdctl get /takbis/postgres-cluster/sync
```


### Config bilgisini almak için 
bu komut ile patroni için etcd de kayıtlı tüm bilgiler alınır.
```bash
etcdctl get /takbis/postgres-cluster/config
```
