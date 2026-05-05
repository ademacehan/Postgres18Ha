### Tüm cluster üyelerinin listelenmesi
```bash
patronictl -c /etc/patroni/patroni.yml list
```

### Topology bilgisinin alınması
```bash
patronictl -c /etc/patroni/patroni.yml topology
```


### Sağlık durumunun görüntülenmesi
```bash 
curl -v http://localhost:8008/health
```

### patroni cluster durmunun anlık olarak monitör edilmesi
```bash 
watch -n 1 "patronictl -c /etc/patroni/patroni.yml list"
```