### Cluster adını öğrneme
```bash
 etcdctl get / --prefix --keys-only
```


### Cluster genel durum görüntüleme
```bash
etcdctl get /takbis/postgres-cluster/status
```

### Clyster içindeki tüm node lar için ayrı ayrı bilgi 
```bash
etcdctl get /takbis/postgres-cluster/members --prefix | grep -E "node[0-9]|xlog_location"
```

### Genel Cluster durumunun görüntülenmesi 
```bash
etcdctl endpoint status --cluster -w table
```

### Sağlık durumunun Görüntülenmesi
```bash
etcdctl endpoint health --cluster -w table
```
|ENDPOINT|HEALTH|TOOK|ERROR|
|--|--|--|--|
|http://172.20.38.17:2379|true|2.241277ms|
|http://172.20.38.19:2379|true|3.044372ms|
|http://172.20.38.18:2379|true|3.196738ms|

  

waatch ile monitor etmek için 
```bash 
watch -n 5 "etcdctl endpoint health --cluster -w table"
```

Sağlıklı bir etcd cluster da took değeri 10ms altında **OLMALIDIR**.
