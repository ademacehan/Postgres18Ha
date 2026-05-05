### Swappiness ###
Sistemin bellek yetersizliği durumunda ne kadar agresif bir şekilde swap alanını kullanacağını belirler.  


***Değer Aralığı:***  
vm.swappiness 0 ile 100 arasında bir değer alabilir.
* 0: Swap kullanımını minimumda tutar.  
Sistem, fiziksel RAM dolana kadar swap alanını kullanmaktan kaçınır.  
* 100: Swap kullanımını maksimumda tutar.  
Sistem, bellek kullanımını dengelemek için swap alanını daha agresif bir şekilde kullanır.  

### 1 Değerinin etkisi
***Minimum Swap Kullanımı:***  
Sistemin swap alanını kullanmaktan büyük ölçüde kaçınmasını sağlar.  
Bu, fiziksel RAM'in daha fazla kullanılmasına ve swap alanının yalnızca çok gerekli olduğunda kullanılmasına neden olur.

***Performans İyileştirmesi:***  
Swap alanı, disk üzerinde olduğu için RAM'e göre çok daha yavaştır.  
Bu nedenle, swap kullanımını minimumda tutmak, bellek erişim hızını artırabilir ve genel sistem performansını iyileştirir.

***Bellek Yoğun Uygulamalar:***  
Bellek yoğun uygulamalar çalıştıran sistemlerde,  
RAM'in daha verimli kullanılmasını sağlar ve disk I/O yükünü azaltır.

***/etc/sysctl.conf***    
dosyası içinde düzenlenmelidir.

```bash
sudo sysctl -p
```

etkin hale gelir.
