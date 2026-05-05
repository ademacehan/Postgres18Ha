# Max open files Control. 

```bash
cat /proc/sys/fs/file-max
sysctl fs.file-max
cat /proc/$(pgrep haproxy | head -n 1)/limits | grep "Max open files"
```

Max Open files Tanımlama. 
```bash
sudo mkdir -p /etc/systemd/system/haproxy.service.d/
vi /etc/systemd/system/haproxy.service.d/override.conf
```  
Override.conf içerisine  
[Service]  
LimitNOFILE=100000  
eklenir.

bu işlemlerden sonra
```bash 
systemctl daemon-reload
systemctl restart rsyslog
systemctl restart haproxy.service
```


sysctl.conf içine
```bash
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.ip_local_port_range = 10000 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_tw_buckets = 262144
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.netfilter.nf_conntrack_max = 262144
net.netfilter.nf_conntrack_tcp_timeout_established = 3600
``` 

parametreleri eklenir.  
```bash
sudo sysctl -p
```
ile aktif edilir.

```bash
vi /etc/modprobe.d/conntrack.conf
```
options nf_conntrack hashsize=65536

değeri eklenir