# ETCD Health Check işlemleri

### Üye bilgilerinin Alınması
#### member(üye) listesinin görüntülenmesi
```bash 
etcdctl member list
```

#### cluster sağlık durumunun kontrol edilmesi
```bash
etcdctl endpoint health --cluster
```
çıktı örneği  
>http://172.20.38.14:2379 is healthy: successfully committed proposal: took = 2.059197ms  
>http://172.20.38.16:2379 is healthy: successfully committed proposal: took = 2.933553ms  
>http://172.20.38.15:2379 is healthy: successfully committed proposal: took = 2.342929ms  


#### cluster durumunun detalı gösterilmesi
```bash
etcdctl endpoint status --cluster -w table
```

|ENDPOINT|ID|VERSION|STORAGE VERSION|DB SIZE|IN USE| PERCENTAGE NOT IN USE |QUOTA|IS LEADER|IS LEARNER|RAFT TERM|RAFT INDEX|RAFT APPLIED INDEX|ERRORS|DOWNGRADE TARGET VERSION|DOWNGRADE ENABLED|
|--|--|--|--|--|--|---|--|--|--|--|--|--|--|--|--|
| http://172.20.38.16:2379 | 35e9b779ffae5306 |   3.6.0 |           3.6.0 |  1.4 MB | 1.4 MB |                    0% |   0 B |     false |      false |         7 |      12005 |              12005 |        |                          |             false |
| http://172.20.38.14:2379 | 3bceaa203ec9eddf |   3.6.0 |           3.6.0 |  1.4 MB | 1.4 MB |                    0% |   0 B |      true |      false |         7 |      12005 |              12005 |        |                          |             false |
| http://172.20.38.15:2379 | f9208abb1f34d7f3 |   3.6.0 |           3.6.0 |  1.4 MB | 1.4 MB |                    0% |   0 B |     false |      false |         7 |      12005 |              12005 |        |                          |             false |


#### Üyeler arasında mesaj gönderme ve alma
***Mesaj Gönderme***

```bash
etcdctl put testkey "hello etcd"
```

***Mesajın Okunması tüm üyelerde test edilmelidir.***

```bash
etcdctl get testkey
```



### Cluster Yük testi

```bash
for i in $(seq 1 10000); do etcdctl put key_$i value_$i; done
```

Yük işlemleri sonrasında cluster hashkv değerinin gözden geçirilmesi

```bash
etcdctl endpoint hashkv
```

### Yavaş disk tespiti

```bash
fio --name=etcd-test \
    --directory=/var/lib/etcd \
    --rw=write \
    --bs=4k \
    --size=256m \
    --ioengine=sync \
    --fdatasync=1
```

cluster içerisinde çökme olursa diks sorunu vardır olmazsa yoktur.

### snapshot restore testi
#snapshot testi yapabilmek için endpoints environment variable kaldırılmalalıdır.

```bash
unset ETCDCTL_ENDPOINTS
```

snapshot alınması
```bash
etcdctl --endpoints=http://172.20.38.15:2379 snapshot save etcd-snap.db
```

### Corruption Injection Testi (Güvenlik/Dayanıklılık Testi)

#Bu test için  datadır bozularak test yapılır.

```bash 
mv /var/lib/etcd/member/snap/db /var/lib/etcd/member/snap/db.bak
systemctl restart etcd
```
#sistem ilgili sunucuda başlamamalı ama diğer iki sunucu devam edebilmelidir.







