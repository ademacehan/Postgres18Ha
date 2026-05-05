

```bash 
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" destination address="172.20.38.21" port port="5000" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" destination address="172.20.38.21" port port="5001" protocol="tcp" accept'
sudo firewall-cmd --reload

```
```bash 
sudo chmod 755 /etc/keepalived/notify.sh
sudo chown root:root /etc/keepalived/notify.sh
```

```bash
systemctl enable keepalived.service
systemctl start keepalived.service
```