# Домашнее задание 12
## Триггеры, поддержка заполнения витрин

Для выполнения домашнего задания использую скрипт [hw_triggers.sql](https://disk.yandex.ru/d/l70AvknAepIJXQ) с предложенным набором таблиц, в скрипте была ошибка в строке установки пути поиска схем search_path, обрезан хвост строки - исправлено:
```sql
-- ДЗ тема: триггеры, поддержка заполнения витрин

DROP SCHEMA IF EXISTS pract_functions CASCADE;
CREATE SCHEMA pract_functions;

SET search_path = pract_functions, public;

-- товары:
CREATE TABLE goods
(
    goods_id    integer PRIMARY KEY,
    good_name   varchar(63) NOT NULL,
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
);
INSERT INTO goods (goods_id, good_name, good_price)
VALUES 	(1, 'Спички хозяйственные', .50),
		(2, 'Автомобиль Ferrari FXX K', 185000000.01);

-- Продажи
CREATE TABLE sales
(
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    good_id     integer REFERENCES goods (goods_id),
    sales_time  timestamp with time zone DEFAULT now(),
    sales_qty   integer CHECK (sales_qty > 0)
);

INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);

-- отчет:
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;

-- с увеличением объёма данных отчет стал создаваться медленно
-- Принято решение денормализовать БД, создать таблицу
CREATE TABLE good_sum_mart
(
	good_name   varchar(63) NOT NULL,
	sum_sale	numeric(16, 2)NOT NULL
);

-- Создать триггер (на таблице sales) для поддержки.
-- Подсказка: не забыть, что кроме INSERT есть еще UPDATE и DELETE

-- Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?
-- Подсказка: В реальной жизни возможны изменения цен.
```

Выполняю скрипт в базе данных
```
sudo -u postgres psql --cluster 18/otus -f /tmp/hw_triggers.sql
```
<img width="1449" height="521" alt="image" src="https://github.com/user-attachments/assets/da1250be-a9da-439f-a69c-89aeebc44af2" /><br>

Создаю триггерную функцию `fnc_good_sum_mart()` и триггер `trg_good_sum_mart` на таблицу `sales` в схеме `pract_functions`
```sql
SET search_path = pract_functions, public;

CREATE OR REPLACE FUNCTION fnc_good_sum_mart()
RETURNS trigger AS
$$
BEGIN
    CASE TG_OP
        WHEN 'DELETE' THEN
          UPDATE good_sum_mart
          SET sum_sale = sum_sale - OLD.sales_qty * (SELECT good_price FROM goods WHERE goods_id = OLD.good_id)
          WHERE good_name = (SELECT good_name FROM goods WHERE goods_id = OLD.good_id);

        WHEN 'UPDATE' THEN
          UPDATE good_sum_mart
          SET sum_sale = sum_sale + (NEW.sales_qty - OLD.sales_qty) * (SELECT good_price FROM goods WHERE goods_id = NEW.good_id)
          WHERE good_name = (SELECT good_name FROM goods WHERE goods_id = NEW.good_id);

        ELSE BEGIN
          UPDATE good_sum_mart
          SET sum_sale = sum_sale + NEW.sales_qty * (SELECT good_price FROM goods WHERE goods_id = NEW.good_id)
          WHERE good_name = (SELECT good_name FROM goods WHERE goods_id = NEW.good_id);

          IF NOT FOUND THEN
            INSERT INTO good_sum_mart (good_name, sum_sale)
            SELECT g.good_name, SUM(g.good_price * NEW.sales_qty)
            FROM goods g
            WHERE g.goods_id = NEW.good_id
            GROUP BY g.good_name;
          END IF;
        END;
    END CASE;
    
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE TRIGGER trg_good_sum_mart
AFTER INSERT OR UPDATE OR DELETE ON sales
FOR EACH ROW
EXECUTE FUNCTION fnc_good_sum_mart();
```

<img width="2105" height="1301" alt="image" src="https://github.com/user-attachments/assets/1dba3771-150d-4d10-a97b-cf5db1caece1" /><br>

Инициализирую таблицу `good_sum_mart` исходными данными продаж
```sql
SET search_path = pract_functions, public;

INSERT INTO good_sum_mart
SELECT G.good_name, sum(G.good_price * S.sales_qty)
FROM goods G
INNER JOIN sales S ON S.good_id = G.goods_id
GROUP BY G.good_name;

SELECT * FROM good_sum_mart;

        good_name         |   sum_sale
--------------------------+--------------
 Автомобиль Ferrari FXX K | 185000000.01
 Спички хозяйственные     |        65.50
(2 строки)
```
<img width="1065" height="641" alt="image" src="https://github.com/user-attachments/assets/05e782d0-61e2-4c64-a79b-cdcb705a0240" /><br>

```sql

-- Добавляю две продажи товара 'Автомобиль Ferrari FXX K' в таблицу sales, проверяю результат

INSERT INTO sales (good_id, sales_qty) VALUES (2, 2);

SELECT * FROM good_sum_mart;

        good_name         |   sum_sale
--------------------------+--------------
 Спички хозяйственные     |        65.50
 Автомобиль Ferrari FXX K | 555000000.03
(2 строки)


-- Правлю добавленную строку `sales_id = 15` на три продажи, проверяю результат

postgres=# SELECT * FROM sales;
 sales_id | good_id |          sales_time           | sales_qty
----------+---------+-------------------------------+-----------
        1 |       1 | 2026-02-27 22:58:23.096612+00 |        10
        2 |       1 | 2026-02-27 22:58:23.096612+00 |         1
        3 |       1 | 2026-02-27 22:58:23.096612+00 |       120
        4 |       2 | 2026-02-27 22:58:23.096612+00 |         1
       15 |       2 | 2026-02-28 01:49:39.937299+00 |         2
(5 строк)

UPDATE sales SET sales_qty = 3 where sales_id = 15;

SELECT * FROM good_sum_mart;

        good_name         |   sum_sale
--------------------------+--------------
 Спички хозяйственные     |        65.50
 Автомобиль Ferrari FXX K | 740000000.04
(2 строки)


-- Удаляю добавленную строку, проверяю результат

DELETE FROM sales where sales_id = 15;

postgres=# SELECT * FROM good_sum_mart;

        good_name         |   sum_sale
--------------------------+--------------
 Спички хозяйственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
(2 строки)
```
<img width="1065" height="1181" alt="image" src="https://github.com/user-attachments/assets/97071052-21ea-47fa-984e-8d3377cbc992" /><br>

```sql

-- Добавляю новый товар 'Компьютер' в таблицу goods с ценой 100 000

INSERT INTO goods (goods_id, good_name, good_price)
VALUES  (3, 'Компьютер', 100000);

SELECT * FROM good_sum_mart;

        good_name         |   sum_sale
--------------------------+--------------
 Спички хозяйственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
(2 строки)

-- Добавляю 4 продажи товара 'Компьютер' в таблицу sales, проверяю результат
INSERT INTO sales (good_id, sales_qty) VALUES (3, 4);
INSERT 0 1
postgres=# SELECT * FROM good_sum_mart;
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозяйственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
 Компьютер                |    400000.00
(3 строки)

postgres=# UPDATE goods SET good_price = 33000 WHERE goods_id = 3;
UPDATE 1
postgres=# INSERT INTO sales (good_id, sales_qty) VALUES (3, 3);
INSERT 0 1
postgres=# SELECT * FROM good_sum_mart;
        good_name         |   sum_sale
--------------------------+--------------
 Спички хозяйственные     |        65.50
 Автомобиль Ferrari FXX K | 185000000.01
 Компьютер                |    499000.00
(3 строки)
```

<img width="1097" height="1031" alt="image" src="https://github.com/user-attachments/assets/98ef4c4d-f83d-4c6c-b24c-a734b01a6bf9" /><br>

