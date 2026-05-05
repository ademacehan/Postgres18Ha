
```bash
sudo dnf install -y haproxy keepalived
```

```bash
sudo firewall-cmd --permanent --add-port={5000/tcp,5001/tcp,7000/tcp}
sudo firewall-cmd –reload
```

```bash
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.def
vi /etc/ haproxy/ haproxy.cfg
```

portlara erişim yetkisi verilmesi
```bash 
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" destination address="172.20.38.21" port port="5000" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" destination address="172.20.38.21" port port="5001" protocol="tcp" accept'
```