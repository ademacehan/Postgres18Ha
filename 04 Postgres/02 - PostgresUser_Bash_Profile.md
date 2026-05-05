# İşletim Sİsteminde Postgres Kullanıcısı için bash file edit

ilk önce postgres Kullanıcısına geçiş yapılır.
```bash
su - postgres
```

```bash
vi .bash_profile
```

```config
PGDATA=/data/pg18
export PGDATA

PATH=/usr/pgsql-18/bin:$PATH
export PATH

LD_LIBRARY_PATH=/usr/pgsql-18/lib
export LD_LIBRARY_PATH

export PGHOST=/tmp

MANPATH=/usr/pgsql-18/share/man
export MANPATH
```
