Full yedek alamk için create sh dosyasınnın oluşturulması  

```bash
#!/bin/bash
# clusater1 için full yedek alan script
pgbackrest --stanza=cluster1 --log-level-console=info --type=full --process-max=8 backup
```