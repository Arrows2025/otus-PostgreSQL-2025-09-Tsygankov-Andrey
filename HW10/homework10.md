# Домашнее задание 10
## Работа с индексами

Для выполнения домашнего задания создаю и заполняю таблицу "Справочник грузов" `nsgruz`. Структура таблицы с данными представлена ниже
```sql
CREATE TABLE nsgruz (
	kod_gr bpchar(6) NOT NULL,
	kl_gr bpchar(1) NULL,
	name_gr bpchar(15) NULL,
	name_grn bpchar(18) NULL,
	name_grk bpchar(18) NULL,
	ns_gruz_s bpchar(6) NULL,
	kod_group bpchar(3) NULL,
	kod_gr_etsng bpchar(6) NULL,
	CONSTRAINT xpknsgruz PRIMARY KEY (kod_gr)
);
```
<img width="1447" height="959" alt="image" src="https://github.com/user-attachments/assets/b9c974c1-6b23-4774-8992-7231577b99c4" /><br><br>

1️⃣ <b>Обычный индекс</b>

Проверяю план запроса с поиском по имени груза - поле `name_grn`, получаю последовательное сканирование по всей таблице Seq Scan со стоимостью выполнения `cost=0.00..155.18`. Создаю обычный индекс на поле `name_grn` и снова проверяю план запроса по имени груза, после создания индекса в плане запроса получаю сканирование индекса Index Scan со стоимостью выполнения `cost=0.28..8.30`, которое быстрее последовательного сканирования без индекса более чем в 18 раз, но незначительно выросла подготовка к выполнению запроса с нуля до 0.28

```
postgres=# explain select * from nsgruz where name_grn = 'БЕНЗИН';
                        QUERY PLAN
----------------------------------------------------------
 Seq Scan on nsgruz  (cost=0.00..155.18 rows=1 width=117)
   Filter: (name_grn = 'БЕНЗИН'::bpchar)
(2 строки)

postgres=# CREATE INDEX idx_nsgruz_name_grn ON nsgruz(name_grn);
CREATE INDEX

postgres=# explain select * from nsgruz where name_grn = 'БЕНЗИН';
                                     QUERY PLAN
------------------------------------------------------------------------------------
 Index Scan using idx_nsgruz_name_grn on nsgruz  (cost=0.28..8.30 rows=1 width=117)
   Index Cond: (name_grn = 'БЕНЗИН'::bpchar)
(2 строки)
```
<img width="1379" height="693" alt="image" src="https://github.com/user-attachments/assets/8b3f4ebc-e70f-4b80-b9f4-e3961228df06" /><br><br>

2️⃣ <b>Индекс для полнотекстового поиска</b>

Проверяю план запроса по короткому имени груза - поле `name_gr` с функцией to_tsvector, которая позволяет преобразовать текст в формат tsvector, оптимизированный для полнотекстового поиска, получаю последовательное сканирование по всей таблице Seq Scan со стоимостью выполнения `cost=0.00..1451.51`. Создаю инвертированный индекс GIN (Generalized Inverted Index) для полнотекстового поиска на поле `name_gr` и снова проверяю план запроса с полнотекстовым поиском по короткому имени груза, после создания индекса в плане запроса получаю сканирование кучи битовой карты Bitmap Heap Scan со стоимостью выполнения `cost=12.95..77.15`, в котором начальная стоимость `cost=0.00..12.95` потрачена на первичное сканирование по индексу с построением битовой карты Bitmap Index Scan, в итоге общий выигрыш стоимости выполнения более чем в 18 раз

```
postgres=# explain select * from nsgruz
where to_tsvector('russian', name_gr) @@ to_tsquery('russian', 'БЕНЗИН');
                                       QUERY PLAN
-----------------------------------------------------------------------------------------
 Seq Scan on nsgruz  (cost=0.00..1451.51 rows=26 width=117)
   Filter: (to_tsvector('russian'::regconfig, (name_gr)::text) @@ '''бензин'''::tsquery)
(2 строки)

postgres=# CREATE INDEX idx_nsgruz_name_gr ON nsgruz USING gin(to_tsvector('russian', name_gr));
CREATE INDEX

postgres=# explain select * from nsgruz
where to_tsvector('russian', name_gr) @@ to_tsquery('russian', 'БЕНЗИН');
                                            QUERY PLAN
---------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on nsgruz  (cost=12.95..77.15 rows=26 width=117)
   Recheck Cond: (to_tsvector('russian'::regconfig, (name_gr)::text) @@ '''бензин'''::tsquery)
   ->  Bitmap Index Scan on idx_nsgruz_name_gr  (cost=0.00..12.95 rows=26 width=0)
         Index Cond: (to_tsvector('russian'::regconfig, (name_gr)::text) @@ '''бензин'''::tsquery)
(4 строки)
```
<img width="1635" height="813" alt="image" src="https://github.com/user-attachments/assets/c1f67899-5fa9-43e9-9a9f-50b8c4558efb" /><br><br>

3️⃣ <b>Индекс на часть таблицы</b>

Проверяю план запроса с поиском по первому классу груза `kl_gr = '1'`, получаю последовательное сканирование по всей таблице Seq Scan со стоимостью выполнения `cost=0.00..155.18`. Создаю индекс на часть таблицы для грузов первого класса по условию `where kl_gr = '1'` и снова проверяю план запроса с поиском по первому классу груза `kl_gr = '1'`, после создания индекса в плане запроса получаю сканирование кучи битовой карты Bitmap Heap Scan со стоимостью выполнения `cost=11.18..109.39`, в котором начальная стоимость `cost=0.00..11.04` потрачена на первичное сканирование по индексу с построением битовой карты Bitmap Index Scan. В итоге быстрее примерно в полтора раза, чем последовательное сканирование по всей таблице. Если запросить данные со вторым классом груза `kl_gr = '2'`, то получу снова последовательное сканирование по всей таблице Seq Scan.

```
postgres=# explain select * from nsgruz where kl_gr = '1';
                         QUERY PLAN
------------------------------------------------------------
 Seq Scan on nsgruz  (cost=0.00..155.18 rows=577 width=117)
   Filter: (kl_gr = '1'::bpchar)
(2 строки)

postgres=# CREATE INDEX idx_nsgruz_kl_gr_1 ON nsgruz(kl_gr) where kl_gr = '1';
CREATE INDEX

postgres=# explain select * from nsgruz where kl_gr = '1';
                                     QUERY PLAN
------------------------------------------------------------------------------------
 Bitmap Heap Scan on nsgruz  (cost=11.18..109.39 rows=577 width=117)
   Recheck Cond: (kl_gr = '1'::bpchar)
   ->  Bitmap Index Scan on idx_nsgruz_kl_gr_1  (cost=0.00..11.04 rows=577 width=0)
(3 строки)

postgres=# explain select * from nsgruz where kl_gr = '2';
                         QUERY PLAN
-------------------------------------------------------------
 Seq Scan on nsgruz  (cost=0.00..155.18 rows=1193 width=117)
   Filter: (kl_gr = '2'::bpchar)
(2 строки)
```
<img width="1395" height="933" alt="image" src="https://github.com/user-attachments/assets/6ac5dc8b-21dc-4366-8550-d4c99a5901b7" /><br><br>

4️⃣ <b>Индекс на поле с функцией</b>

Проверяю план запроса с поиском по сокращённому имени груза с функцией приведения к нижнему регистру lower, получаю последовательное сканирование по всей таблице Seq Scan со стоимостью выполнения `cost=0.00..180.85`. Создаю индекс на поле с функцией lower и снова проверяю план запроса с поиском по полю с функцией, после создания индекса в плане запроса получаю сканирование кучи битовой карты Bitmap Heap Scan со стоимостью выполнения `cost=4.48..62.25`, в котором начальная стоимость `cost=0.00..4.48` потрачена на первичное сканирование по индексу с построением битовой карты Bitmap Index Scan. В итоге быстрее почти в три раза, чем последовательное сканирование по всей таблице.

```
postgres=# explain select * from nsgruz where lower(ns_gruz_s) = 'бензин';
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on nsgruz  (cost=0.00..180.85 rows=26 width=117)
   Filter: (lower((ns_gruz_s)::text) = 'бензин'::text)
(2 строки)

postgres=# CREATE INDEX idx_nsgruz_ns_gruz_s ON nsgruz(lower(ns_gruz_s));
CREATE INDEX

postgres=# explain select * from nsgruz where lower(ns_gruz_s) = 'бензин';
                                     QUERY PLAN
------------------------------------------------------------------------------------
 Bitmap Heap Scan on nsgruz  (cost=4.48..62.25 rows=26 width=117)
   Recheck Cond: (lower((ns_gruz_s)::text) = 'бензин'::text)
   ->  Bitmap Index Scan on idx_nsgruz_ns_gruz_s  (cost=0.00..4.48 rows=26 width=0)
         Index Cond: (lower((ns_gruz_s)::text) = 'бензин'::text)
(4 строки)
```
<img width="1395" height="753" alt="image" src="https://github.com/user-attachments/assets/01b2f8f9-8c1b-4673-b0be-72f211b231b6" /><br><br>

5️⃣ <b>Индекс на несколько полей</b>

Проверяю план запроса с поиском по двум полям, получаю последовательное сканирование по всей таблице Seq Scan со стоимостью выполнения `cost=0.00..168.01`. Создаю составной индекс на два поля, которые участвуют в поиске, и снова проверяю план запроса с поиском по двум полям, после создания индекса в плане запроса получаю сканирование кучи битовой карты Bitmap Heap Scan со стоимостью выполнения `cost=18.18..123.67`, в котором начальная стоимость `cost=0.00..17.94` потрачена на первичное сканирование по индексу с построением битовой карты Bitmap Index Scan. В итоге быстрее примерно в 1,36 раза, чем последовательное сканирование по всей таблице

```
postgres=# explain select * from nsgruz where kl_gr = '3' and kod_group = '18';
                            QUERY PLAN
------------------------------------------------------------------
 Seq Scan on nsgruz  (cost=0.00..168.01 rows=966 width=117)
   Filter: ((kl_gr = '3'::bpchar) AND (kod_group = '18'::bpchar))
(2 строки)

postgres=# CREATE INDEX idx_nsgruz_kl_gr_kod_group ON nsgruz(kl_gr, kod_group);
CREATE INDEX
postgres=# explain select * from nsgruz where kl_gr = '3' and kod_group = '18';
                                         QUERY PLAN
--------------------------------------------------------------------------------------------
 Bitmap Heap Scan on nsgruz  (cost=18.18..123.67 rows=966 width=117)
   Recheck Cond: ((kl_gr = '3'::bpchar) AND (kod_group = '18'::bpchar))
   ->  Bitmap Index Scan on idx_nsgruz_kl_gr_kod_group  (cost=0.00..17.94 rows=966 width=0)
         Index Cond: ((kl_gr = '3'::bpchar) AND (kod_group = '18'::bpchar))
(4 строки)
```
<img width="1523" height="723" alt="image" src="https://github.com/user-attachments/assets/7fcdd954-41d6-4b96-abdc-54c8fe9e5b5d" /><br><br>

6️⃣ Размер пяти созданных индексов превысил размер самой таблицы из пяти тысяч записей (запрос взят из практики и сохранён на будущее)
```sql
SELECT
    TABLE_NAME,
    pg_size_pretty(table_size) AS table_size,
    pg_size_pretty(indexes_size) AS indexes_size,
    pg_size_pretty(total_size) AS total_size
FROM (
    SELECT
        TABLE_NAME,
        pg_table_size(TABLE_NAME) AS table_size,
        pg_indexes_size(TABLE_NAME) AS indexes_size,
        pg_total_relation_size(TABLE_NAME) AS total_size
    FROM (
        SELECT ('"' || table_schema || '"."' || TABLE_NAME || '"') AS TABLE_NAME
        FROM information_schema.tables
    ) AS all_tables
    ORDER BY total_size DESC

    ) AS pretty_sizes
WHERE table_name like '%nsgruz%';
```
<img width="822" height="597" alt="image" src="https://github.com/user-attachments/assets/7328bced-5dc3-49d5-994e-f6b0764239db" /><br>

Можно проверить и удалить неиспользуемые индексы, чтобы они не занимали место на диске (запрос взят из практики и сохранён на будущее)
```sql
SELECT s.schemaname,
       s.relname AS tablename,
       s.indexrelname AS indexname,
       pg_size_pretty(pg_relation_size(s.indexrelid)) AS index_size,
       s.idx_scan
FROM pg_catalog.pg_stat_user_indexes s
   JOIN pg_catalog.pg_index i ON s.indexrelid = i.indexrelid
WHERE s.idx_scan < 10      -- has never been scanned
  AND 0 <>ALL (i.indkey)  -- no index column is an expression
  AND NOT i.indisunique   -- is not a UNIQUE index
  AND NOT EXISTS          -- does not enforce a constraint
         (SELECT 1 FROM pg_catalog.pg_constraint c
          WHERE c.conindid = s.indexrelid)
ORDER BY pg_relation_size(s.indexrelid) DESC;
```
<img width="937" height="587" alt="image" src="https://github.com/user-attachments/assets/f323ef0a-90c0-4ecf-9001-703875a74a87" />
