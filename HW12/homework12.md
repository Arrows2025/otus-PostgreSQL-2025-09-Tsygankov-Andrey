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
VALUES 	(1, 'Спички хозайственные', .50),
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




<img width="2105" height="1301" alt="image" src="https://github.com/user-attachments/assets/1dba3771-150d-4d10-a97b-cf5db1caece1" /><br>
