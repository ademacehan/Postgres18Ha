
```bash
sudo semanage port -a -t http_port_t -p tcp 7000
sudo semanage port -a -t http_port_t -p tcp 7001
sudo semanage port -a -t http_port_t -p tcp 7003
sudo semanage port -a -t http_port_t -p tcp 5000
sudo semanage port -a -t http_port_t -p tcp 5001
sudo semanage port -a -t http_port_t -p tcp 5003
sudo semanage port -a -t http_port_t -p tcp 6000
sudo semanage port -a -t http_port_t -p tcp 6001
sudo semanage port -a -t http_port_t -p tcp 6003
sudo semanage port -a -t http_port_t -p tcp 5005
sudo semanage port -a -t http_port_t -p tcp 6432
setsebool -P haproxy_connect_any 1
getenforce
sudo setenforce 0
sudo setenforce 1
```
rsyslog içerisine local2 eklenir.  

```bash
vi /etc/rsyslog.conf
```
rsyslog dosyasında aşağıdak değerler aktif edilir.  
module(load="imudp") # needs to be done just once    
input(type="imudp" port="514")   
local2.*                                                /var/log/haproxy.log



```bash
systemctl daemon-reload
systemctl enable haproxy.service
systemctl daemon-reload
systemctl restart  rsyslog
systemctl start haproxy.service
```

