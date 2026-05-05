# Log dosyalarının silinmesi. 
Log dosyalarını silmek için  
```bash
rm -rf postgresql-04.csv
```

## Log dizini dolduğunda silme işlemi 
```bash
rm -rf postgresql-04.csv
```

Dosya silme işleminin devam edip etmediğinin kontrolü. 
```bash 
sudo lsof |grep deleted
```

Burada   

postgres  1261446                         postgres    8w      REG              253,3 19519823872        132 /pglog/postgresql-01.csv (deleted).  
8w ile linux bu dosyaya hala 8. kanaldan yazmaya devam ettiğini gçösterir.  
Yazma işlemini sonlandırmak için. 

```bash
sudo sh -c 'true > /proc/1261446/fd/8'
````
proc/: Burası Linux'un "beyni" gibidir. Çalışan tüm süreçlerin (process) bilgilerini canlı olarak burada tutar.
/1261446/:  Postgres sürecinin kimlik numarası (PID).
/fd/8: Postgres'in "8 numaralı açık kapısına doğrudan müdahale et" demektir.
true >: true komutu hiçbir çıktı üretmez (boştur). > işareti ise "hedefteki dosyaya boşalt" demektir.

Log dosyasını rotate etmek için   
```bash
sudo -u postgres psql -c "SELECT pg_rotate_logfile();"
```



