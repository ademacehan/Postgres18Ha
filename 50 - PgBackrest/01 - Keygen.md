# Tüm sunucularda keygen create edilir.
Tüm patroni node pgbacrest ve backup recover sunucusunda
```bash 
ssh-keygen -t ed25519
```

keygen diğer saunuculara kopyalanır.  
```bash
ssh-copy-id postgres@172.20.38.12
```

# Tüm sunculardan birbirlerine geçiş testi yapılır.
```bash
ssh postgres@172.20.38.12
```


## geçmiş tanımlı tüm ssh-keygen leri silme
```bash
rm -rf ~/.ssh/id_ed25519* ~/.ssh/authorized_keys ~/.ssh/known_hosts
rm -rf ~/.ssh/id_rsa* ~/.ssh/id_ed25519*
```