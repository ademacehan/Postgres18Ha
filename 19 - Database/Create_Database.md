### Türkçe veri tabanı create etmek için

Rock linux işletim sistemin TR dil paketinde veri tabanı create etmek için öncesin de
```bash
sudo dnf install langpacks-tr
```


```bash 
create database adem  
    with encoding='UTF-8' LC_COLLATE = 'C' 
          LC_CTYPE = 'tr_TR.utf8' 
    template= template0;
```