# ETCD Kurulumunun sıfırlanması

```bash
systemctl stop etcd
pkill -9 etcd
systemctl disable etcd
rm -rf /var/lib/etcd/*
rm -rf /etc/systemd/system/etcd.service.d/
systemctl daemon-reload
systemctl enable --now etcd
systemctl start etcd
systemctl status etcd
```