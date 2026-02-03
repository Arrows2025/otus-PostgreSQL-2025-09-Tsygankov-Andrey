# Домашнее задание 08
## Механизм блокировок

1️⃣ Настраиваю сервер, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Для этого проверяю и включаю параметр `log_lock_waits`, устанавливаю параметр `deadlock_timeout` равным 200 мс и перезагружаю файлы конфигурации PostgreSQL
```
sudo -u postgres psql --cluster 18/otus

postgres=# show log_lock_waits;
postgres=# ALTER SYSTEM SET log_lock_waits = on;
postgres=# show deadlock_timeout;
postgres=# ALTER SYSTEM SET deadlock_timeout = 200;
postgres=# SELECT pg_reload_conf();
postgres=# show log_lock_waits;
postgres=# show deadlock_timeout;
```
<img width="857" height="1211" alt="image" src="https://github.com/user-attachments/assets/3d99012f-46a2-42cf-92bc-9902e73fceb1" /><br>

Узнаю PID сессии, создаю таблицу test_locks с текстовым полем, заполняю её случайными сгенерированными данными в размере 1 миллион строк
```
sudo -u postgres psql --cluster 18/otus

postgres=# SELECT pg_backend_pid();
postgres=# CREATE TABLE test_locks (
    id SERIAL PRIMARY KEY,
    Data TEXT
);
postgres=# INSERT INTO test_locks (Data) SELECT md5(random()::text) FROM generate_series(1, 1000000);

postgres=# UPDATE test_locks set Data = Data || '+test1';
```

```
postgres=# UPDATE test_locks set Data = Data || '+test2';
```




<img width="1305" height="461" alt="image" src="https://github.com/user-attachments/assets/4cba337a-802d-4ee9-b525-ee71e8cc6afb" />



Задание со ⭐ : Безымянная процедура, которая в цикле несколько раз обновляет все строки и добавляет к строкам номер шага цикла +S1+S2+S3... и т.д.

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

2️⃣ Для инициализации Pgbench создаю для тестов базу данных pbtest и инициализирую утилиту Pgbench

3️⃣

4️⃣
