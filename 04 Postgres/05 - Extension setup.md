
```bash
dnf -y install postgresql-contrib
dnf -y install epel-release
dnf config-manager --enable crb
crb enable
dnf -y install postgis36_18
```

```bash
dnf install -y postgresql18-plpython3
```


### timescale 
```bash 
 dnf search timescaledb
 dnf install -y timescaledb_18.x86_64
```

```bash
sudo dnf install -y timescaledb-2-postgresql-18
###repoda 10 versiyonu 9 a çwkilir
sudo sed -i 's/\$releasever/9/g' /etc/yum.repos.d/pgdg-redhat-all.repo
```

```bash
sudo dnf clean all
sudo dnf makecache
dnf search timescaledb | grep postgresql-18
```

```bash
sudo dnf install -y timescaledb-2-postgresql-18
veya
sudo dnf install -y timescaledb_18
```

```bash
sudo sed -i 's/rhel-9/rhel-\$releasever/g' /etc/yum.repos.d/pgdg-redhat-all.repo 
```

```bash
sudo dnf install -y nfs-utils pgbackrest
```

```bash
sudo dnf install -y pgaudit_18
```