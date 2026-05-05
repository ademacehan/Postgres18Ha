### En son revision numarasının alınması
```bash 
rev=$(etcdctl --endpoints=http://127.0.0.1:2379 endpoint status --write-out=json | egrep -o '"revision":[0-9]*' | cut -d: -f2)
```

### revision'a kadar olan eski verileri temizlenmesi
```bash 
etcdctl --endpoints=http://127.0.0.1:2379 compact $rev
```

## Defragmentasyon
### Defragmentasyon tüm üyelere
etcd, eski ve gereksiz verileri düzenli olarak siler (bu işleme Compaction denir).   
Ancak, bu silme işlemi veritabanı dosyasında (disk üzerinde) boşluklar, yani "parçalar" (fragment) bırakır.   
Tıpkı bir bilgisayarın hard diskinde olduğu gibi,   
etcd veritabanı da zamanla parçalanır (fragmente olur).

etcdctl defrag komutu bu boşlukları doldurur,   
kullanılmayan alanı temizler ve veritabanı dosyasını yeniden düzenleyerek disk üzerindeki gerçek boyutunu küçültür .  

--cluster kullanımı tüm üyelerde çalıştıracaktır.   
Bu komut esnasında lock işlemi yaparak geçici olarak okuma yazma işlemlerini yapamaz.
```bash
etcdctl defrag --cluster
```

#### Defragmentasyon rolling
defrag işlemi rolling yapılarak daha sağlıklı bir şekilde yapılır.  
önce leader olmayan kullanıcılardan başlanır.  

```bash
etcdctl defrag --endpoints="http://172.20.38.14:2379"
etcdctl defrag --endpoints="http://172.20.38.16:2379"
etcdctl defrag --endpoints="http://172.20.38.15:2379"
```


### Defragmentation (Diskteki Boşluğu Geri Kazanma)
```bash 
etcdctl --endpoints=http://127.0.0.1:2379 defrag
```