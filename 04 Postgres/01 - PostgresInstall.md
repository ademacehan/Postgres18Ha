# Postgres Veri tabanı için paketlerin kurulması

```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-10-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

İşlertim sistem,i ile gelen paketin disable edilmesi
```bash
sudo dnf -qy module disable postgresql
```

Postgres Veri Tabanının install edilmesi
```bash
sudo dnf install -y postgresql18-server postgresql18-contrib
```


