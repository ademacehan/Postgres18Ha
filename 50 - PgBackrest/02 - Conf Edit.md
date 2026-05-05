### Merkezi sunucuda conf edit
/etc/pgbackrest.conf
```bash
[global]
repo1-path=/pgbackrest/
repo1-retention-full=2
repo1-retention-diff=16
repo1-retention-archive-type=full
repo1-retention-archive=7
log-level-file=info
process-max=8
compress-type=lz4
buffer-size=16MiB


[cluster1]
pg1-path=/data/pg18
pg1-host=172.20.38.11
pg2-path=/data/pg18
pg2-host=172.20.38.12
pg3-path=/data/pg18
pg3-host=172.20.38.13

```

```bash
ssh-keygen -t ed25519
```

```bash
cat ~/.ssh/id_ed25519.pub
```

aynı şekilde diğer sunucularda üretilen keygenler ana yedek sunucusuna da eklenir.