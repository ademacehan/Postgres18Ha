
#### Tüm cluster üyelerinin listelenmesi
```bash
patronictl -c /etc/patroni/patroni.yml list
```

#### yaml değişiklerinin cluster içine uygulanması
```bash
patronictl -c /etc/patroni/patroni.yml reload  postgres-cluster
```
#### reinit 
reinit işlemi lag oluşmuş veya sekron bozulmuş cluster üyesi için yapılır.
```bash
patronictl -c /etc/patroni/patroni.yml reinit postgres-cluster node1
```

#### Sağlık durumunun görüntülenmesi
```bash 
curl -v http://localhost:8008/health
```

#### patroni cluster durmunun anlık olarak monitör edilmesi
```bash 
watch -n 1 "patronictl -c /etc/patroni/patroni.yml list"
```