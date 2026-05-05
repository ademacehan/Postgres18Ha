Herhangi bir servis için default dosya okuma yetkisi  
genellikle **1024** dür.

Eğer pgbouncer havuz boyutu 1024 den fazla olacaksa bunun için  
tanımlama yapılması gerekektedir.

ilk Önce systemd altında pgbouncer için service dizini create edilir.
```bash
mkdir -p /etc/systemd/system/pgbouncer.service.d/
```

dizin içerisine kernel dosya okuma kapasitesinin yaılacağı dosya create edilir.

```bash
vi /etc/systemd/system/pgbouncer.service.d/limits.conf
``` 

dosya içerisine 
```bash
[Service]
LimitNOFILE=2000
``` 

2000 değeri pgouncer pool değerinden az olmamalıdır.


Service yeninden başlatılır.
```bash
systemctl daemon-reload
systemctl restart pgbouncer.service
```

deamon değişikliği için servis işletim sistemi tarafında yeniden başlatılalıdır.