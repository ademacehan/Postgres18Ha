# Tüm sunucularda birbirleri arasında geçiş için SSH-KEYGEN 

Her sunucuda aşağıdaki işlem ayrı ayrı yapılacaktır.

```bash
ssh-keygen -t rsa -b 4096
```

tüm uyarılar enter ile geçilir.
Daha sonra her kullanıcıya login olunarak ssh copy yapılacaktır.
```bash
ssh-copy-id root@db02
```

```bash
ssh-copy-id root@db03
```

```bash
ssh-copy-id root@ha01
```

```bash
ssh-copy-id root@ha02
```

```bash
ssh-copy-id root@backup01
```

```bash
ssh-copy-id root@backup02
```

db01 sunucusunda yapılan işlem diğer sunucularda da tek tek yapılması gerekmektedir.
