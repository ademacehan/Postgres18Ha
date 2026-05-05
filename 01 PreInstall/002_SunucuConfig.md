# Sunucu isimlerinin belirlenmesi

```bash
sudo hostnamectl set-hostname db01
```
her sunucuda kendisi için ayrıca çalıştırılacaktır.

# etc/hosts dosyasının yapılandırılması

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4

172.20.38.14 db01
172.20.38.15 db02
172.20.38.16 db03

172.20.38.17 ha01
172.20.38.18 ha02

172.20.38.11 backup01
172.20.38.12 backup02
172.20.38.13 backup03

172.20.38.19 cqrs

172.20.38.21 vip

