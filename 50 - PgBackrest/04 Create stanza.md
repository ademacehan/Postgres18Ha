ilk Önce stanza create edilir.
```bash
pgbackrest --stanza=cluster1 stanza-create
```


full backup
```bash
pgbackrest --stanza=cluster1 --log-level-console=info --type=full --process-max=8 backup
```
