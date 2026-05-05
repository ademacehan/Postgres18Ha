## TCP Ayarları:
net.core.somaxconn = 8192  
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_tw_reuse = 2

net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

### net.core.netdev_max_backlog
Network kartı (NIC) paketleri kernel’e teslim ettiğinde,  
CPU yetişemezse bu paketlerin geçici olarak bekletileceği kuyruk boyutunu belirler.
CPU yetişemezse en fazla değer kadar paket paket kernel receive backlog’unda tutulabilir.
### net.ipv4.tcp_fin_timeout
Bağlantı kapatılırken karşı tarafın FIN paketine cevap verdikten sonra socket’in   
FIN_WAIT2 durumunda kaç saniye bekleyeceğini belirler.   
Varsayılan değer kullanılabilir.

***Bu parametre şu durumlarda kritik olur:***    
	•	Çok yüksek network throughput (10GbE / 25GbE)  
	•	Çok yüksek PPS (packet per second)  
	•	API gateway arkasında DB  
	•	DDoS / SYN flood  
	•	Mikroservis patlaması  

***PostgreSQL sisteminde:***  
	•	Büyük veri akışı yoksa  
	•	1GbE ağ varsa  
	•	Günlük 10K bağlantı ama düşük eşzamanlılık varsa   
*****DEFAULT***** değer yeterlidir.  
	•	1000+ concurrent connection  
	•	Çok yoğun kısa süreli bağlantı  
	•	Çok fazla küçük paket (API trafiği)  
varsa artırmak gerekir.

***Çok Düşükse Ne Olur?***  
CPU o an meşgulse:  
	•	Paketler drop edilir  
	•	netstat -s içinde “dropped packets” artar  
	•	Latency spike oluşur  

***Çok Yüksek Olursa?***    
	•	Kernel memory tüketimi artar    
	•	Ama 16384 modern sunucu için sorun değildir    

***KONTROL İŞLEMLERİ***  
```bash

```
### net.ipv4.tcp_keepalive_time
Bir TCP bağlantısı idle (hiç veri akışı yok) kaldıktan sonra kaç saniye sonra ilk keepalive paketi gönderileceğini belirler.  
TCP keepalive:  
	•	Karşı taraf hâlâ yaşıyor mu?  
	•	Network bağlantısı kopmuş mu?  
	•	Firewall bağlantıyı düşürmüş mü?  
bunu anlamak için gönderilen küçük kontrol paketidir.  

bağlantı 300sn (5dk)  idle kalırsa kernel karşı taraf keepalive paketi gönderir.
Eğer cevap gelmezse:  
	•	tcp_keepalive_intvl aralıklarla tekrar dener  
	•	tcp_keepalive_probes kadar cevap alamazsa  
	•	Bağlantıyı düşürür  

Değerin düşük tutulması gereksiz cpu yükü yapar.
### net.ipv4.tcp_tw_reuse  
**timewait** durumundaki soketlerin kullanılmasını kontrol eder.    
Bu süre boyunca aynı 4-tuple (src ip, src port, dst ip, dst port) tekrar kullanılamaz.    
Yoğun bağlantı churn olan sistemlerde (PgBouncer, API, microservice) TIME_WAIT birikmesi port exhaustion’a yol açabilir.
tcp_tw_reuse = 1 TIME_WAIT socket’ler tüm outbound bağlantılar için reuse edilir.  
tcp_tw_reuse = 2 sadece aynı makine içi bağlantılarda çalışır.    
postgres ve bouncer aynı sunucuda ise 2 değilse 1
bunun için ayar yapma durum kontrolü
```bash
ss -ant | grep TIME-WAIT | wc -l
```
### net.core.rmem_max
Bir soket için kullanılabilecek maksimum alıcı tampon boyutunu belirler.
**Yüksek Bant Genişliği:**  
Yüksek bant genişliğine sahip ağlarda, daha büyük alıcı tamponlar,  
daha fazla verinin daha hızlı bir şekilde alınmasına olanak tanır.  
**Veri Akışı:**  
Büyük veri akışları sırasında paket kaybını ve gecikmeyi azaltır.  
### net.core.wmem_max
Bir soket için kullanılabilecek maksimum gönderici tampon boyutunu belirler.  
**Yüksek Performans:**  
Büyük gönderici tamponlar,   
daha fazla verinin daha hızlı bir şekilde gönderilmesine olanak tanır.
**Veri Transferi:**  
Büyük veri transferleri sırasında daha iyi performans sağlar.
### net.ipv4.tcp_rmem  
TCP bağlantıları için alıcı tampon boyutlarının minimum, varsayılan ve maksimum değerlerini belirler.
### net.ipv4.tcp_wmem   
TCP bağlantıları için gönderici tampon boyutlarının minimum, varsayılan ve maksimum değerlerini belirler.

*** Ağ için**   
4Kb açılış için küçük paketlerin iletilmesinde mantıklı bir değerdir.
87KB Ortalama büyüklükteki  veri için uygundur.
16MB Büyük paketler iin yeteri kadar büyüktür ve veri iletiminde büyük fayda sağlayacaktır.
Daha büyük değerler için **ram** kaynağı gözden geçirilmelidir.


### Değerleri kontrol etmek için 
```bash
sysctl net.core.somaxconn
sysctl net.ipv4.tcp_max_syn_backlog
sysctl net.core.netdev_max_backlog
sysctl net.ipv4.tcp_fin_timeout
sysctl net.ipv4.tcp_keepalive_time
sysctl net.ipv4.tcp_tw_reuse
sysctl net.core.rmem_max
sysctl net.core.wmem_max
sysctl net.ipv4.tcp_rmem
sysctl net.ipv4.tcp_wmem
```

```bash
cat /proc/sys/net/core/somaxconn
cat /proc/sys/net/ipv4/tcp_max_syn_backlog
cat /proc/sys/net/core/netdev_max_backlog
cat /proc/sys/net/ipv4/tcp_fin_timeout
cat /proc/sys/net/ipv4/tcp_keepalive_time
cat /proc/sys/net/ipv4/tcp_tw_reuse
cat /proc/sys/net/core/rmem_max
cat /proc/sys/net/core/wmem_max
cat /proc/sys/net/ipv4/tcp_rmem
cat /proc/sys/net/ipv4/tcp_wmem
```

### Değerlerin anlık runtinme değiştirilmesi
```
sudo sysctl -w net.core.somaxconn=8192
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=8192
sudo sysctl -w net.core.netdev_max_backlog = 16384
sudo sysctl -w net.ipv4.tcp_fin_timeout=15
sudo sysctl -w net.ipv4.tcp_keepalive_time=120
sudo sysctl -w net.ipv4.tcp_tw_reuse = 2
sudo sysctl -w net.core.rmem_max = 16777216
sudo sysctl -w net.core.wmem_max = 16777216
sudo sysctl -w net.ipv4.tcp_rmem = 4096 87380 16777216
sudo sysctl -w net.ipv4.tcp_wmem = 4096 65536 16777216
```