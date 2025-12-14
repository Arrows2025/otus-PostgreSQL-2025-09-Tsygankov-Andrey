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

Добавляю в конец файла `postgresql.conf` параметры настройки PostgreSQL из прикрепленного к материалам занятия файла, с скобках для себя сохранил базовые значения параметров
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


Задание со :star: Безымянная процедура, которая в цикле несколько раз обновляет все строки и добавляет к строкам номер шага цикла +S1+S2+S3... и т.д.
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


<img width="1602" height="1258" alt="image" src="https://github.com/user-attachments/assets/20d639ea-a351-411b-b1c9-56109331a5ea" />
