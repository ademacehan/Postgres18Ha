20 merkezi pgbacrest sunucusunda üretilen keygen tüm sunucularda postgres kullanıcısı ile 
oluşturulur.
```bash
echo "üretilen ssh key" >> /var/lib/pgsql/.ssh/authorized_keys
```

tüm clientlarda pgbacrest conf edit işlemi yapılır.
```bash
vi /etc/pgbackrest.conf
```

paroni config de aşağıdaki ayarlar mutlaka yapılmış olmalıdır.
```bash
archive_command: pgbackrest --stanza=cluster1 archive-push %p
archive_mode: 'on'
```


```bash
[global]
#arşiv sunucu bilgisi path bilgisinin girilmesi
repo1-host=172.20.38.20
repo1-host-user=postgres
repo1-path=/data/pgbackrest_storage

[cluster1]
pg2-path=/data/pg18
```