# statement_timeout

Bu parametere ile belirlenen süreyi aşan tüm sorgular otomatik kesilecektir.  
**instance, veri tabanı ve kullanıcı seviyesinde verilebilir.**

## İnstance seviyesinde timeout 
<b>postgresql.conf </b> dosyasının içersinde eklenebilir.  
```bash 
statement_timeout = '1min'
``` 

fakat bu tanım çok büyük riskler içermektedir.
Örneğin vacuum, primary key create index create gibi

## Veri Tabanı seviyesinde timeout 

```bash
ALTER DATABASE db_adi SET statement_timeout = '1min';
```

Bu kullanımda instance seviyesindeki ayar kadar sorun teşkil edebilir.  
Bir veri tabanında bu parametre geçilirse o veri tabanındaki index, constraint ve vacuum gibi işlemler süreyi aşarsa kesilecektir.

## Kullanıcı seviyesinde timeout
Kullanıcı seviyesinde timeout en güvenli yoldur. Sadece o kullanıcıda bu kısıt çalışacaktır.  
Diğer kullanıcıların ve admin işlemlerini etkilemeyecektir.

Belirtilen kullanıcı artık verilen süreyi aşan sürede sorgu atamayacaktır.

```bash
ALTER ROLE app_user SET statement_timeout = '1min';
```

**statement_timeout** yetkisi gruptan inheritence ile alınmaz.

### Kullanıcı yetkisinin verildiğinin kontrolü

```bash
SELECT rolname, rolconfig 
FROM pg_roles 
WHERE rolname = 'app_user'
```


