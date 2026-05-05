### dirty_background_ratio
Bu ayar, sistem belleğinde kirli (henüz diske yazılmamış) sayfaların oranını belirler.  
Bu oran aşıldığında arka plan yazma işlemlerinin başlatılmasını sağlar.

***Kirli Sayfalar:***    
Bellekteki kirli sayfalar, henüz diske yazılmamış veri içeren bellek sayfalarıdır.  
Bu sayfalar, dosya sistemine yapılan yazma işlemleri sırasında oluşur.


***Oran:***   
vm.dirty_background_ratio, toplam sistem belleğinin yüzde kaçı kadar kirli sayfa olabileceğini belirler.  
Bu oran aşıldığında, arka plan yazma işlemleri başlatılır ve kirli sayfalar diske yazılır.

***Ram boyutu ve disk yazma hızına göre belirlenmelidir.***


***/etc/sysctl.conf***    
dosyası içinde düzenlenmelidir.

```bash
sudo sysctl -p
```

etkin hale gelir.