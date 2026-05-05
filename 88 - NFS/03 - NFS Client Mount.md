ilk önce mount klasörü oluşturulur.
```bash 
 mkdir pgbackrest_storage
 sudo chown  postgres:postgres /data/pgbackrest_storage/
```

fstab dosyasına nfs dizini mount edilir.
```bash
vi /etc/fstab
```

```bash
172.20.38.20:/data/pgbackrest_storage /data/pgbackrest_storage nfs defaults,_netdev,soft,intr,timeo=14,retrans=2 0 0
```

Bağlantı testi
```bash
systemctl daemon-reload
sudo mount -a
df -h | grep pgbackrest
```

Yetki verilmesi
```bash
 sudo chown  postgres:postgres /data/pgbackrest_storage/
```

selinux izinlerinin verilmesi
```bash
sudo setsebool -P virt_use_nfs 1
```