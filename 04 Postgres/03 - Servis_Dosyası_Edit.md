# Postgres servis dosyasının edit edilmesi

```bash
vi /usr/lib/systemd/system/postgresql-18.service
```

Servis dosyası içerisindeki 
Environment değeri bash file içerisine eklediğimiz değer ile aynı hale gertirilir.

Buradaki değer postgres veri tabanının kurulacağı dizindir.
```config
# Note: avoid inserting whitespace in these Environment= lines, or you may
# break postgresql-setup.

# Location of database directory
Environment=PGDATA=/data/pg18/
```