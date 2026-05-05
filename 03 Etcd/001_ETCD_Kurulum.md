# ETCD Kurulum

## İşletim Sistemine ETCD Kullanıcısının Eklenmesi

```bash
sudo useradd --system --home /var/lib/etcd --shell /bin/false etcd
```

```bash
cd /usr/local/src 
git clone -b v3.6.0 https://github.com/etcd-io/etcd.git
cd etcd
./scripts/build.sh
export PATH="$PATH:`pwd`/bin"
mkdir - p /etc/etcd
```

## ETCD konfigurasyonunun yapılması

```bash
sudo vi /etc/etcd/etcd.conf
```
```config
name: db01
data-dir: /var/lib/etcd
quota-backend-bytes: 8589934592
auto-compaction-retention: "1"


listen-client-urls: http://127.0.0.1:2379,http://172.20.38.11:2379
advertise-client-urls: http://172.20.38.13:2379

listen-peer-urls: http://172.20.38.13:2380
initial-advertise-peer-urls: http://172.20.38.13:2380

initial-cluster-token: patroni-cluster-etcd
initial-cluster: db01=http://172.20.38.11:2380,db02=http://172.20.38.12:2380,db03=http://172.20.38.13:2380
initial-cluster-state: new


```

```bash
chown -R etcd:etcd /etc/etcd
```

Src içerisindeki etcd bin dosyaları local/bin altına taşınır.

```bash
sudo cp /usr/local/src/etcd/bin/etcd /usr/local/bin/
sudo cp /usr/local/src/etcd/bin/etcdctl /usr/local/bin/
sudo cp /usr/local/src/etcd/bin/etcdutl /usr/local/bin/
sudo chmod +x /usr/local/bin/etcd*
```

Servis Dosyasının yapılandırılması 

```bash
sudo nano /etc/systemd/system/etcd.service
```

```config
[Unit]
Description=Etcd Server db01
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name db01 \
  --data-dir /var/lib/etcd \
  --listen-client-urls http://127.0.0.1:2379,http://10.0.0.101:2379 \
  --advertise-client-urls http://10.0.0.101:2379 \
  --listen-peer-urls http://10.0.0.101:2380 \
  --initial-advertise-peer-urls http://10.0.0.101:2380 \
  --initial-cluster db01=http://10.0.0.101:2380,db02=http://10.0.0.102:2380,db03=http://10.0.0.103:2380 \
  --initial-cluster-token patroni-cluster-etcd \
  --initial-cluster-state new \
  --auto-compaction-mode=periodic \
  --auto-compaction-retention=1 \
  --quota-backend-bytes=8589934592 \
  --heartbeat-interval=100 \
  --election-timeout=1000 \
  --snapshot-count=10000

# Yeniden Başlatma Ayarları
Restart=on-failure
RestartSec=5s

# Kaynak Limitleri
LimitNOFILE=65536
LimitMEMLOCK=infinity

# Performans ve Öncelik Ayarları
Nice=-10
CPUSchedulingPolicy=fifo
CPUSchedulingPriority=1
IOSchedulingClass=realtime
IOSchedulingPriority=0

[Install]
WantedBy=multi-user.target
```
***heartbeat-interval :*** Bu parametre etcd master'ın diğer etcd düğümlere **ben hala liderim** mesajını gönderdiği zaman sıklığıdır.  
Default 100ms dir.  Bu süresnin seçiminde en etkili yol iki sunucu arasındaki RTT süresidir.
RTT = Round trip time  
RTT süresi hesaplamada genel olarak ping testi kullanılır.  
Fakat burada ping tek başına etkili olmayacaktır.  Etcd'nin bu veriyi diske yazma süreside eklenmelidir.  
fio --name=etcd-test \
    --directory=/var/lib/etcd \
    --rw=write \
    --bs=4k \
    --size=256m \
    --ioengine=sync \
    --fdatasync=1  
fio komutu ile paket yazma bilgisi elde edilebilir.  
Örnek :  
| 99.00th=[ 3785], 99.50th=[ 4883], 99.90th=[ 7963], 99.95th=[ 9372],
Burda yazma işlemi 3.7 ms bitmiş.  
ping 0.3 ms olsun.  
Cpu zamanı da muhtemel 2 ms dir.  
Toplam  RTT süresi 6-7 ms olacağı için  default 100 kullanılabilir.  
heartbeat için RTT süresinin 1.5 kat kadarı da kullanılabilir. RTT 8 değil de 100 çıksa idi. heartbeat değerine 250'ye kadar değer girilebilir.
Çok az değer vermek network trafiği ve cpu tüketimi yapacağı için default değer altında değer verilmemesi önerilir.  
Default üstü için hesaplama yapılması önerilir.

***election-timeout :*** yeni lider seçmek için beklenecek süre  
bu süre heartbeat değerinin 5 ila 10 katı arasında set edilmesi önerilmektedir.
50 sn den büyük olmamalıdır.

***snapshot-count :*** Etcd, her snapshot-count kadar işlemden sonra bellekteki veriyi diske yazar. Varsayılan değer (100.000) dir.  
Bazen çok büyük log dosyalarına ve yazma anında performans düşüşüne neden olabilir.  
Bu prametrenin değerinin 10.000 olarak güncellenmesi tuning olafak önerilmektedir.
eğer hala etcd memory vd cpu kullanımı yüksek ise bu değeri 
5000 olarak dosyada güncelleyip çalışan etcd de değiştirmek için   
etcd --snapshot-count=5000  
komutu kullannılmalıdır.



```bash
sudo mkdir -p /var/lib/etcd
export ETCDCTL_API=3
sudo systemctl daemon-reload
sudo systemctl enable --now etcd
systemctl status etcd
```

