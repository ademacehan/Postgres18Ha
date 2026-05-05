
/etc/exports 
içerisine paylaşılacak dizin kayıt edilir.
ilk önce dizin daha sonra erişim izni verilecek sunucu ip adresi ve erişim hakları belirtilir..
```bash 
/pgbackrest 172.20.38.11(rw,sync,no_root_squash) 172.20.38.12(rw,sync,no_root_squash) 172.20.38.13(rw,sync,no_root_squash)
```

Firewall yetkilerinin verilmesi
```bash
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload

sudo systemctl enable --now nfs-server
sudo exportfs -ra
```