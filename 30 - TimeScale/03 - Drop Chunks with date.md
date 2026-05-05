## Tarih bazlı chunk verilerin temizlenmesi
```bash
SELECT drop_chunks('public.process_log_work_log', '2026-05-03'::date);
```