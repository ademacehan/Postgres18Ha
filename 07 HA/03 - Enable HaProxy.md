
```bash
sudo semanage port -a -t http_port_t -p tcp 7000
sudo semanage port -a -t http_port_t -p tcp 5001
sudo semanage port -a -t http_port_t -p tcp 5001
sudo semanage port -a -t http_port_t -p tcp 6432
setsebool -P haproxy_connect_any 1
getenforce
sudo setenforce 0
sudo setenforce 1
vi /etc/rsyslog.d/haproxy.log
```

```bash
local2.*  /var/log/haproxy.log
```

```bash
systemctl daemon-reload
systemctl enable haproxy.service
systemctl daemon-reload
systemctl restart  rsyslog
systemctl start haproxy.service
```