## Hyper Table oluşturma
1. parametre : Tablo schema.tablo_adi
2. timescale için kullanılacak  timestampz formatındaki tarih alanı 
3. ne kadar zaman aralığında bölünlenecek.
4. 
```bash

SELECT create_hypertable('public.outbox_log_archive', 'created_date', chunk_time_interval => INTERVAL '1 day');
```
