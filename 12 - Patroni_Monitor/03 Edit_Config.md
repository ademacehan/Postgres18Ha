### Patroni Config Edit
```bash
patronictl -c /etc/patroni/patroni.yml edit-config
```
Vi formatinda bir düzenleme ekranı açılacaktır.  
Gerekli düzenleme yapıldıktan sonra 
":wq"
ile değişiklikler kayıt et çık işlemi yapılır.

Yapılan değişiklikler ekrana yansıtılır,

Örnek 
((venv) ) postgres@TK-TAKBIS-PREPROD-CLS03-DB01 ~ # patronictl -c /etc/patroni/patroni.yml edit-config
---
+++
@@ -23,7 +23,7 @@
     datestyle: iso, mdy
     default_text_search_config: pg_catalog.english
     dynamic_shared_memory_type: posix
-    effective_cache_size: 5GB
+    effective_cache_size: 6GB
     effective_io_concurrency: 200
     enable_mergejoin: true
     escape_string_warning: true

Apply these changes? [y/N]:

Burada yes denildiğinde değişiklik tüm cluster daki sunuculara yansıtılacaktır.

### Mevcut config dosyasının yedeklenmesi
```bash
patronictl -c /etc/patroni/patroni.yml show-config > patroni.yml.20260104
```

patroni config etcd içerisinde zaten yedekli fakat tedbir için bir dosyada da yedek olsun denilirse
show-config komutu ile başka bir dosyaya yazdırılabilmektedir.