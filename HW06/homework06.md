# Домашнее задание 06
## Настройка autovacuum с учётом особенностей производительности

Для выполнения домашнего задания использую уже установленный на виртуальной машине сервер Ubuntu 24.04.3 и установленную базу данных PostgreSQL 18 с базовыми параметрами. Настраиваю процессор витруальной машины на 2 ядра и ставлю 4 Гб оперативной памяти.

Для инициализации Pgbench создаю для тестов базу данных `pbtest` и инициализирую утилиту Pgbench
```
sudo -u postgres psql

postgres=# CREATE DATABASE pbtest;
CREATE DATABASE
postgres=# \q

sudo su postgres
pgbench -i pbtest
```

<img width="2009" height="791" alt="image" src="https://github.com/user-attachments/assets/082ed790-7350-45df-bc0e-0b34ab5b050b" /><br>


Запускаю Pgbench на базовых параметрах БД PostgreSQL
```
pgbench -c 8 -P 6 -T 60 -U postgres pbtest
```
<img width="1321" height="851" alt="image" src="https://github.com/user-attachments/assets/cb2485a5-8c4c-4564-9a8c-2f2cd430af7b" /><br>

Добавляю в конец файла `postgresql.conf` параметры настройки PostgreSQL из прикрепленного к материалам занятия файла, в скобках для себя сохранил базовые значения параметров
```
max_connections = 40 (default 100)
shared_buffers = 1GB (default 16384)
effective_cache_size = 3GB (default 524288)
maintenance_work_mem = 512MB (default 65536)
checkpoint_completion_target = 0.9 (default 0.9)
wal_buffers = 16MB (default 512)
default_statistics_target = 500 (default 100)
random_page_cost = 4 (default 4)
effective_io_concurrency = 2 (default 16)
work_mem = 6553kB (default 4096)
min_wal_size = 4GB (default 80)
max_wal_size = 16GB (default 1024)
```
Перестартовываю кластер PostgreSQL `main`
```
sudo nano /etc/postgresql/18/main/postgresql.conf
sudo pg_ctlcluster 18 main restart
sudo pg_ctlcluster 18 main status
pg_lsclusters
```
<img width="2121" height="371" alt="image" src="https://github.com/user-attachments/assets/2b7d9954-f7ec-4e75-ad60-9d5ec954a4d3" /><br>

Повторно запускаю Pgbench на новых параметрах БД PostgreSQL
```
pgbench -c 8 -P 6 -T 60 -U postgres pbtest
```
<img width="1321" height="881" alt="image" src="https://github.com/user-attachments/assets/76f716c3-b135-4d0e-8494-528c02ace5e2" /><br>

Тест Pgbench с новыми параметрами не показал изменения производительности в транзакциях в секунду, так как почти все изменённые параметры относились к оперативной памяти, что не являвляется узким местом производительности и повышения производительности на виртуальных машинах не даёт

Создаю таблицу `test_table` с текстовым полем, заполняю её случайными сгенерированными данными в размере 1 миллион строк и узнаю её размер - 87 MB
```
sudo -u postgres psql

postgres=# CREATE TABLE test_table (
    id SERIAL PRIMARY KEY,
    Data TEXT
);
CREATE TABLE
postgres=# INSERT INTO test_table (Data) SELECT md5(random()::text) FROM generate_series(1, 1000000);
INSERT 0 1000000
postgres=# SELECT pg_size_pretty(pg_total_relation_size('test_table'));
 pg_size_pretty
----------------
 87 MB
(1 строка)
```
<img width="1657" height="581" alt="image" src="https://github.com/user-attachments/assets/f0d69721-3a63-446a-80a2-9105239f1164" /><br>


Задание со :star: : Безымянная процедура, которая в цикле несколько раз обновляет все строки и добавляет к строкам номер шага цикла +S1+S2+S3... и т.д.
```
DO
$BODY$
BEGIN
    FOR i IN 1 .. 10 LOOP
        UPDATE test_table set Data = Data || '+S' || i;
        RAISE NOTICE 'Шаг %', i;
    END LOOP;
END;
$BODY$
LANGUAGE plpgsql;
```
Запускаю цикл на пять шагов с 1-го по 5-ый и пять раз обновляю все строки таблицы `test_table`
```
DO
$BODY$
BEGIN
    FOR i IN 1 .. 5 LOOP
        UPDATE test_table set Data = Data || '+S' || i;
        RAISE NOTICE 'Шаг %', i;
    END LOOP;
END;
$BODY$
LANGUAGE plpgsql;
```
<img width="1118" height="784" alt="image" src="https://github.com/user-attachments/assets/706c60ec-f5ac-4421-913f-71bd1d079b23" /><br>

После завершения работы цикла проверяю размер таблицы (523 MB), количество мёртвых строчек (5 000 000 строк) и когда последний раз запускался autovacuum - в 20:44
```
SELECT pg_size_pretty(pg_total_relation_size('test_table')); -- 523 MB
```
```
SELECT relname,
       n_live_tup,
       n_dead_tup,
       trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%",
       last_autovacuum
  FROM pg_stat_user_tables
  WHERE relname = 'test_table';
```

<img width="893" height="245" alt="image" src="https://github.com/user-attachments/assets/d2d698e1-f57d-47c5-8f9c-d9e3d5eb03b6" /><br>

После запуска autovacuum в 20:45 все мёртвые строки удалены, размер таблицы не изменился - 523 MB

<img width="885" height="308" alt="image" src="https://github.com/user-attachments/assets/e9218f33-d6d2-4be1-9db8-c5acf815a27b" /><br>

Отключаю autovacuum на таблице `test_table` и запускаю цикл ещё на пять шагов с 6-го по 10-ый и снова пять раз обновляю все строки таблицы `test_table`
```
ALTER TABLE test_table SET (autovacuum_enabled = off);
```
```
DO
$BODY$
BEGIN
    FOR i IN 6 .. 10 LOOP
        UPDATE test_table set Data = Data || '+S' || i;
        RAISE NOTICE 'Шаг %', i;
    END LOOP;
END;
$BODY$
LANGUAGE plpgsql;
```

<img width="1120" height="783" alt="image" src="https://github.com/user-attachments/assets/a48c467d-b156-4661-92e2-811edcbbd1aa" /><br>

Размер таблицы после повторной подобной по количеству изменений процедуры 610 MB, вырос несильно, так как было использовано свободное место в таблице, которое освободил autovacuum. Количество мёртвых строчек так же 5 000 000 строк

<img width="611" height="214" alt="image" src="https://github.com/user-attachments/assets/ff8bf7c8-a8d1-41ee-a6b9-7258f1afdb9b" /><br>

Теперь запускаю цикл на 10 шагов с 11-го по 20-ый и десять раз обновляю все строки таблицы `test_table`
```
DO
$BODY$
BEGIN
    FOR i IN 11 .. 20 LOOP
        UPDATE test_table set Data = Data || '+S' || i;
        RAISE NOTICE 'Шаг %', i;
    END LOOP;
END;
$BODY$
LANGUAGE plpgsql;
```
<img width="1203" height="810" alt="image" src="https://github.com/user-attachments/assets/17a54088-a509-41bf-8b91-7f28ed7a88bd" /><br>

Размер таблицы увеличился втрое - 1814 MB, так как из-за отключённого autovacuum к неочищенным мёртвым строкам в 5 миллионов добавилось ещё 10 миллионов мёртвых строк

<img width="734" height="206" alt="image" src="https://github.com/user-attachments/assets/e88ef1d9-18c9-4da3-9c4f-ab06b56c730b" /><br>

Включаю autovacuum на таблицу `test_table`. После запуска autovacuum 15 миллионов мёртвых строк почистились, размер таблицы не изменился - 1814 MB
```
ALTER TABLE test_table SET (autovacuum_enabled = on);
```

Запускаю `vacuum full`, чтобы освободить место на диске после проведённого эксперимента, размер таблицы `test_table` после завершения `vacuum full` - 156 MB

<img width="934" height="319" alt="image" src="https://github.com/user-attachments/assets/fef44772-a40c-440e-b0f3-a2ea2e2cb3b4" />
