# Домашнее задание 10
## Работа с индексами

Для выполнения домашнего задания создаю и заполняю таблицу "Справочник грузов" `nsgruz`, структура таблицы представлена ниже
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

1️⃣ Обычный индекс
Проверяю план запроса с поиском по имени груза - поле `name_grn`, получаю последовательное сканирование по всей таблице Seq Scan со стоимостью выполнения `cost=0.00..155.18`. Создаю обычный индекс на поле `name_grn` и снова проверяю план запроса по имени груза, после создания индекса в плане запроса получаю сканирование индекса Index Scan со стоимостью выполнения `cost=0.28..8.30`, быстрее последовательного сканирования без индекса более чем в 18 раз, но незначительно выросла подготовка к выполнению запроса с нуля до 0.28

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
<img width="1379" height="693" alt="image" src="https://github.com/user-attachments/assets/8b3f4ebc-e70f-4b80-b9f4-e3961228df06" />
