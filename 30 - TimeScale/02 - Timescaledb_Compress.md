## Tabloda compress

Compression kuralı ekleme
```bash

ALTER TABLE public.outbox_log_archive SET (
  timescaledb.compress,
  timescaledb.compress_segmentby = 'index_name',
  timescaledb.compress_orderby = 'created_date DESC'
);
  
SELECT add_compression_policy('public.outbox_log_archive'::regclass, INTERVAL '2 days');
```

Compression kuralı kaldırma

```bash 

SELECT remove_compression_policy('public.outbox_log_archive');
```


## Compress yapılmş chunk veri boyutu görüntüleme

***compress yapılmış chunk veri boyutu Ram %25 ini geçmemelidir.***

```bash 

SELECT chunk_schema, chunk_name, compression_status, before_compression_total_bytes, after_compression_total_bytes, node_name
FROM chunk_compression_stats('outbox_log_archive')
ORDER BY chunk_name DESC
LIMIT 5;
```

## Süre beklemeden compress işlemi

```bash

DO $$
DECLARE
    chunk_reg regclass;
BEGIN
    -- Tüm chunk'ları dön, sadece sıkışmamış olanları sıkıştır
    FOR chunk_reg IN 
        SELECT (quote_ident(chunk_schema) || '.' || quote_ident(chunk_name))::regclass
        FROM timescaledb_information.chunks
        WHERE hypertable_name = 'outbox_log_archive'
    LOOP
        BEGIN
            RAISE NOTICE 'İşlemdeki parça: %', chunk_reg;
            PERFORM compress_chunk(chunk_reg);
            RAISE NOTICE 'Başarıyla sıkıştırıldı: %', chunk_reg;
        EXCEPTION 
            WHEN others THEN
                -- Eğer zaten sıkıştırılmışsa veya başka bir hata varsa atla
                RAISE NOTICE 'Atlanıyor (Muhtemelen zaten sıkışık): %', chunk_reg;
        END;
    END LOOP;    
    RAISE NOTICE 'İşlem tamamlandı.';
END $$;
```