Patroni için gerekli dizinlerin create edilmesi  
ve postgres kullanıcısına yetki verilmesi

```bash
sudo mkdir -p /var/log/patroni
sudo chown postgres:postgres /var/log/patroni
sudo chmod 750 /var/log/patroni
sudo setenforce 0
sudo semanage fcontext -a -t var_log_t "/var/log/patroni(/.*)?"
sudo restorecon -R /var/log/patroni
sudo mkdir -p /opt/patroni
sudo chown postgres:postgres /opt/patroni
```

Postgres kullanıcısına geçilir.  
Patroni için python env set edilir.
```bash
sudo su - postgres
```

```bash
python3 -m venv /opt/patroni/venv 
```

```bash
python3 --version
source /opt/patroni/venv/bin/activate
```

```bash
pip install --upgrade pip
pip install patroni[etcd3] psycopg2-binary
/opt/patroni/venv/bin/patroni --version
```

Tekrar root kullanıcısına geçilir.
```bash
exit
sudo mkdir /etc/patroni
sudo chown postgres:postgres /etc/patroni
```

