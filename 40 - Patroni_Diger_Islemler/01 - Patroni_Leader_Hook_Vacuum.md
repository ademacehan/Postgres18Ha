## Kullanıcının oluşturulması 

```bash 
create role db_vacuum_role with login;
grant connect on database ekolog to db_vacuum_role;
```

```bash
\c ekolog
```

```bash
grant maintain on public.outbox_process_log to db_vacuum_role;
grant maintain on public.outbox_log to db_vacuum_role;
```

## hba kullanıcı tanımı 

```bash
 pg_hba:
  - local   ekolog          db_vacuum_role  trust
  - local   all             all             peer
```
## Leader Hook
Patroni hook ile sadece master da çalışması gereken timer gerektiren görevlerin ayarlanması

### edit config

yaml dosya içerisinde rol değişim kontrolü bash dosyasının eklenmesi
```bash 

callbacks:
  on_role_change: /data/bash_files/on_role_change.sh
```

### on_role_change.sh 

```bash
#!/bin/bash
# $1 = role (leader / replica)
# $2 = cluster name

ROLE="$1"

if [ "$ROLE" = "leader" ]; then
    echo "$(date) Became LEADER, enabling vacuum job"

    # örnek: systemd timer başlat
    systemctl start vacuum-users.timer

elif [ "$ROLE" = "replica" ]; then
    echo "$(date) Became REPLICA, disabling vacuum job"

    systemctl stop vacuum-users.timer
fi
```
```bash 
chmod +x /data/bash_files/on_role_change.sh
```

### vacuum-users.service 
/etc/systemd/system/vacuum-users.service
```bash 
[Unit]
Description=Vacuum users table
After=postgresql.service

[Service]
Type=oneshot
User=postgres
ExecStart=psql -d ekolog -U db_vacuum_role -c "VACUUM (ANALYZE, PARALLEL 4) public.outbox_process_log;"
ExecStart=psql -d ekolog -U db_vacuum_role -c "VACUUM (ANALYZE, PARALLEL 4) public.outbox_log;"
```

### vacuum-users.timer
/etc/systemd/system/vacuum-users.timer

```bash
[Unit]
Description=Run vacuum users every 30 minutes

[Timer]
OnBootSec=10min
OnUnitActiveSec=30min

[Install]
WantedBy=timers.target
```

```bash
systemctl daemon-reload
systemctl enable vacuum-users.timer
```

