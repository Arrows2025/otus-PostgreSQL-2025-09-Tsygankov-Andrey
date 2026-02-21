# Домашнее задание 11
## Секционирование таблицы

Для выполнения домашнего задания разворачиваю [демонстрационную базу данных](https://postgrespro.ru/education/demodb) для СУБД PostgreSQL, которая в качестве предметной области использует авиаперевозки, выбираю объём данных по полётам за год - https://edu.postgrespro.ru/demo-20250901-1y.sql.gz
```
gunzip -c /tmp/demo-20250901-1y.sql.gz | psql -U postgres --cluster 18/otus
```
<img width="1849" height="1241" alt="image" src="https://github.com/user-attachments/assets/10453d9f-b351-4969-8191-0a927606a5d5" />
...
<img width="1849" height="1241" alt="image" src="https://github.com/user-attachments/assets/90c85f06-547c-4a57-b67f-eef6aec6d331" /><br><br>

Проанализировав схему данных, объёмы данных в таблицах и скорость выполнения запросов, выбираю для секционирования таблицу бронирований `bookings`, так как она является основной сущностью схемы данных, а выборка по диапазону дат бронирования требует значительного времени и ресурсов базы данных
<img width="1572" height="1235" alt="image" src="https://github.com/user-attachments/assets/53f23af1-c79e-48d5-bfbe-db90c46110a0" />

Некоторые таблицы по объёму превосходят таблицу бронирований `bookings`, но выборки из них в логике схемы данных производятся по ключевым полям, выполняются быстро и не требуют значительных ресурсов

<img width="1054" height="348" alt="image" src="https://github.com/user-attachments/assets/19a13d33-eb11-4927-9ac0-dc5326b4696a" /><br>

Секционирование таблицы бронирований целесообразно производить по диапазону дат бронирования - поле `book_date`. По аналогии с оригинальной таблицей бронирований `bookings` создаю секционированную таблицу бронирований `bookings_part`, в состав первичного ключа добавляю поле секционирования `book_date`, так как ограничение уникальности в секционированной таблице должно включать все секционирующие столбцы
```sql
CREATE TABLE bookings.bookings (
	book_ref bpchar(6) NOT NULL,
	book_date timestamptz NOT NULL,
	total_amount numeric(10, 2) NOT NULL,
	CONSTRAINT bookings_pkey PRIMARY KEY (book_ref)
);
```
```sql
\connect demo

CREATE TABLE bookings.bookings_part (
	book_ref bpchar(6) NOT NULL,
	book_date timestamptz NOT NULL,
	total_amount numeric(10, 2) NOT null,
	CONSTRAINT bookings_part_pkey PRIMARY KEY (book_ref,book_date)
) PARTITION BY RANGE (book_date);
```
Таблица бронирований `bookings` содержит данные за год с сентября 2025 года по август 2026 года включительно
```sql
SELECT min(book_date), max(book_date) FROM bookings.bookings; -- 2025-09-01 00:00:06.265219+00 | 2026-08-31 23:59:58.283465+00
```
<img width="2217" height="581" alt="image" src="https://github.com/user-attachments/assets/9b30763d-7daa-4234-a2e6-a1b3677a7c9f" /><br>

Создам 12 секций с диапазонами по месяцам и секцию `default` для данных, которые не попадают в диапазоны секционирования
```sql
CREATE TABLE bookings.bookings_part_2025_09 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2025-09-01') TO ('2025-10-01');
CREATE TABLE bookings.bookings_part_2025_10 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');
CREATE TABLE bookings.bookings_part_2025_11 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2025-11-01') TO ('2025-12-01');
CREATE TABLE bookings.bookings_part_2025_12 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2025-12-01') TO ('2026-01-01');
CREATE TABLE bookings.bookings_part_2026_01 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');
CREATE TABLE bookings.bookings_part_2026_02 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
CREATE TABLE bookings.bookings_part_2026_03 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2026-03-01') TO ('2026-04-01');
CREATE TABLE bookings.bookings_part_2026_04 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2026-04-01') TO ('2026-05-01');
CREATE TABLE bookings.bookings_part_2026_05 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2026-05-01') TO ('2026-06-01');
CREATE TABLE bookings.bookings_part_2026_06 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2026-06-01') TO ('2026-07-01');
CREATE TABLE bookings.bookings_part_2026_07 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2026-07-01') TO ('2026-08-01');
CREATE TABLE bookings.bookings_part_2026_08 PARTITION OF bookings.bookings_part FOR VALUES FROM ('2026-08-01') TO ('2026-09-01');

CREATE TABLE bookings.bookings_part_other PARTITION OF bookings.bookings_part DEFAULT;
```
<img width="2217" height="851" alt="image" src="https://github.com/user-attachments/assets/7f50fa4c-1f86-4296-9275-b817b9bd013d" /><br>

Переношу данные из оригинальной таблицы бронирований `bookings` в секционированную таблицу бронирований `bookings_part`, после переноса проверяю распределение данных по секциям - данные распределились равномерно
```sql
demo=# INSERT INTO bookings.bookings_part
SELECT * FROM bookings.bookings;
INSERT 0 4905238

demo=# SELECT tableoid::regclass AS partition, count(*)
FROM bookings_part
GROUP BY tableoid;
       partition       | count
-----------------------+--------
 bookings_part_2025_09 | 448064
 bookings_part_2025_10 | 434159
 bookings_part_2025_11 | 410670
 bookings_part_2025_12 | 410796
 bookings_part_2026_01 | 412036
 bookings_part_2026_02 | 370985
 bookings_part_2026_03 | 408667
 bookings_part_2026_04 | 383124
 bookings_part_2026_05 | 399596
 bookings_part_2026_06 | 403237
 bookings_part_2026_07 | 411392
 bookings_part_2026_08 | 412512
(12 строк)
```
<img width="2217" height="731" alt="image" src="https://github.com/user-attachments/assets/67aa2e8e-653d-45e5-afe3-ae8e039b2c48" /><br>


<img width="2521" height="911" alt="image" src="https://github.com/user-attachments/assets/cbf1f389-d1c4-4504-ba08-0555181e6330" />
