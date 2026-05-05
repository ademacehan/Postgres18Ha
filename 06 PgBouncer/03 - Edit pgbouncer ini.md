```bash 
cd /etc/pgbouncer 
cp pgbouncer.ini pgbouncer.ini.def
> /etc/pgbouncer/pgbouncer.ini
vi /etc/pgbouncer/pgbouncer.ini
```

```bash 
[databases]
aacehan= dbname=aacehan host=172.20.38.15 port=5432
aacehan_read = dbname=aacehan host=172.20.38.15 port=5432
bouncer= dbname=bouncer host=172.20.38.15 port=5432

[pgbouncer]
;;;
;;; Administrative settings
;;;
logfile = /var/log/pgbouncer/pgbouncer.log
pidfile = /var/run/pgbouncer/pgbouncer.pid
;;;
;;; Where to wait for clients
;;;

listen_addr = 0.0.0.0
listen_port = 6432

;;;
;;; Authentication settings
;;; Bu ayar sayesinde diğer kullanıcıların authentication bilgisini 
;;; bouncer kullanıcı yapacaktır.
;;; 
auth_user = bouncer
auth_type = scram-sha-256
auth_dbname = bouncer
auth_file = /etc/pgbouncer/userlist.txt

;;;auth_query = select p_user,p_password from public.lookup($1)
auth_query = SELECT p_user, p_password FROM (SELECT * FROM public.lookup($1)) AS t
;;;
;;; Users allowed into database 'pgbouncer'
;;;
admin_users = aacehan, postgres, bouncer
stats_users = aacehan, postgres, bouncer

;;; pool_modes session transaction statement
pool_mode= transaction

;;;
ignore_startup_parameters = extra_float_digits
max_client_conn = 20000
default_pool_size = 50

verbose = 0

;;; log parameters
log_connections = 1
log_disconnections = 1
```

```bash
vi /etc/pgbouncer/userlist.txt
```
Sadece bouncer kullanıcısı bu dosyaya eklenmelidir.  
!!!Diğer kullanıcıların login bilgisi bouncer kullanıcısı üzerinden doğrulanacaktır.!!!
```bash 
"bouncer" "bouncer2025"
```