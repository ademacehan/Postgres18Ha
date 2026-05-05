# Güvenlikk Duvarının yapılandırılması

```bash
sudo systemctl enable --now firewalld
```

```bash
sudo firewall-cmd --permanent --add-service=ssh
```

```bash
sudo firewall-cmd --permanent --add-port={2379/tcp,2380/tcp,5432/tcp,8008/tcp,6432/tcp}
```

```bash
sudo firewall-cmd --reload
```