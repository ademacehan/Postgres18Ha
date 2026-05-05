## Huge Pages
Huge Pages, bellek yönetimini optimize eder. Aşağıdaki adımları izleyerek Huge Pages etkinleştirilir.  

Linux çekirdeğinde Huge Pages adı verilen büyük bellek sayfalarının sayısını ayarlamak için kullanılır.  
Huge Pages, bellek yönetimini optimize ederek bellek erişim hızını artırır ve bellek parçalanmasını azaltır.  

***Bellek Yönetimi:***  
Huge Pages, standart bellek sayfalarından (genellikle 4 KB) daha büyük (genellikle 2 MB veya daha fazla) bellek sayfaları kullanır.  
Bu, bellek yönetimi için gereken sayfa tablosu girişlerini azaltır ve bellek erişim hızını artırır.  

***Performans Artışı:***   
Büyük bellek sayfaları, bellek erişim gecikmelerini azaltarak ve CPU önbellek kullanımını iyileştirerek performansı artırabilir.  
Özellikle büyük veri tabanı uygulamaları için faydalıdır.

***Bellek Parçalanmasını Azaltma:***   
Huge Pages, bellek parçalanmasını azaltarak bellek tahsisini daha verimli hale getirir.

***PostgreSQL ile Uyum:***  
PostgreSQL, Huge Pages'i destekler ve bu sayede daha iyi performans elde edilebilir.  
PostgreSQL'in huge_pages ayarını on veya try olarak ayarlayarak bu özelliği etkinleştirilir.  

***Mevcut Huge Pages Değerini Öğrenme***  
```bash
cat /proc/sys/vm/nr_hugepages
```

***Huge page Boyutunu Öğrenme***
```bash
grep Hugepagesize /proc/meminfo
```

***Kullanılan ve Serbest Huge Pages:***  
```bash
grep Huge /proc/meminfo
```
AnonHugePages:    110592 kB  
ShmemHugePages:        0 kB  
FileHugePages:         0 kB  
HugePages_Total:   17000  
HugePages_Free:     1545  
HugePages_Rsvd:     1107  
HugePages_Surp:        0  
Hugepagesize:       2048 kB  
Hugetlb:        34816000 kB  

***AnonHugePages:***  
Anonim olarak tahsis edilen Huge Pages miktarını gösterir.  
Bu, genellikle uygulamalar tarafından doğrudan kullanılan bellek miktarıdır.

***ShmemHugePages:***  
Paylaşılan bellek segmentleri için kullanılan Huge Pages miktarını gösterir.

***FileHugePages:***  
Dosya destekli Huge Pages miktarını gösterir. 

***HugePages_Total:***  
Toplam tahsis edilen Huge Pages sayısını gösterir. 

***HugePages_Free:***  
Kullanılabilir durumda olan (boş) Huge Pages sayısını gösterir.  

**HugePages_Rsvd:***  
Rezerv edilmiş ancak henüz kullanılmayan Huge Pages sayısını gösterir. 

***HugePages_Surp:*** 
Fazladan tahsis edilen Huge Pages sayısını gösterir. 

***Hugepagesize:***  
Her bir Huge Page'in boyutunu gösterir. 

***Hugepagesize:***  
Her bir Huge Page'in boyutunu gösterir.

### Huge Page ayarının değiştirilmesi
```bash
echo 32768 > /proc/sys/vm/nr_hugepages
```

***/etc/sysctl.conf***  dosyasına 
```bash
vm.nr_hugepages = 32768 
```
eklenir

### Postgres kullanıcısına HUge Pages Yetkisinin verilmesi
***Sistem Ayarlarını Güncellenmesi:***  
*****/etc/security/limits.conf***** dosyası düzenlenir.  
postgres soft memlock unlimited   
postgres hard memlock unlimited  
Bu ayarlar, PostgreSQL kullanıcısının bellek kilitleme limitlerini kaldırır.  
Böyle Huge Pages kullanacaktır.






