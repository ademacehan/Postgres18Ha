## SHMMAX ve SHMALL Ayarları

### shmmax
İlk önce ***SHMMAX*** değeri set edilir.
***Bu değer Maximum size of shared memory segment (bytes) ***
değerini etkileyeceği için Shared Memory değerine eşit yada daha büyük olmalıdır.
Varsayılan olarak ram değeri yaygın kullanım şeklidir.  
Örnek 128GB ram için hesaplanması:  
128 * 1024 * 1024 * 1024 = 137438953472  
(Gb) (mb)  (kb)   (byte)
```bash
kernel.shmmax = 137438953472  # 128GB
```
!!! Düşük olması durumunda aşağıdaki hatalar alınabilir.  
**FATAL: could not create shared memory segment**  
**FATAL: semget: No space left on device**  
**FATAL: could not fork new process** 


## **shmall**  
SHMMAX değerinin linux page sayısına bölünmesi ile elde edilir.  
SHMMAX/page_size  
```bash
getconf PAGE_SIZE
```
PAGE_SIZE genel de 4096 dır.  
Çıkan sonucun tam sayı değeri alınır.  
137438953472/4096 = 33554432

```bash
echo "kernel.shmall = 33554432" | sudo tee -a /etc/sysctl.d/99-postgresql.conf
```

## **sem**  
kernel.sem = SEMMSL SEMMNS SEMOPM SEMMNI  
4 parameetreden oluşmaktadır.  
### semmsl
Bir set içerisindeki Maximum semaphores sayısıdır. 17 den az olamaz
Bu sayı eş zamanlı çalışabilecek process sayısı demektir.  
Ortalama değer: 250 
sunucunun işlemci ve ram ve sistem ihtiyacına göre değiştirilmelidir.

### SEMMNS
Tüm setlerdeki toplam semaphore sayısı  
ceil((max_connections + autovacuum_max_workers + 5) / 16) * semmsl plus room for other applications  
Sistemdeki max connection sayısı = 1000 
autovacuum_max_worker = 10
( 1000 + 10 + 5) / 16 * 250 = 
64 * 250 = 16000

### SEMOPM
Aynı anda yapılabilecek en fazla semaphore işlemi sayısıdır.
Genelde 32 veya 100 olarak set edilir.
Bunu belirlemede sunucu işlemcisi göz önünde tutulmalıdır.  
semopm = 32  

### SEMMNI
Maximum number of shared memory segments system-wide  
En az 	alabileceği değer 
ceil(num_os_semaphores / 16) * 17 plus room for other applications  
Genelde 1024 olarak set edilir.

```bash
echo "kernel.sem = 250 16000 32 1024" | sudo tee -a /etc/sysctl.d/99-postgresql.conf
```

### file_max
Linux sistem genelinde açılabilecek toplam dosya (file descriptor) limitini belirler.

```bash
echo "fs.file-max = 1000000" | sudo tee -a /etc/sysctl.d/99-postgresql.conf
```

### overcommit_memory 

Bellek aşım tahsisis durumunu belirler.

```bash
echo "vm.overcommit_memory = 2" | sudo tee -a /etc/sysctl.d/99-postgresql.conf
```
0 : Heuristic overcommit  
1: Always  
2: Don't

### overcommit_ratio
Sistemin donanım kapasitesine %80'i aşmayacak şekilde yapılandırılması önerilmektedir.

```bash
echo "vm.overcommit_ratio = 2" | sudo tee -a /etc/sysctl.d/99-postgresql.conf
```
****Avantajları****  
PostgreSQL’de OOM Killer riskini azaltır.   
Patroni cluster’da ani node çökmesinin önüne geçer   
Memory limitleri daha öngörülebilir olur.  

****Dezavantajı****  
Bazı büyük sorgular bellek isteyince   
kernel “RAM yetmez” diyerek:  
Cannot allocate memory   
hatası alınabilir.


