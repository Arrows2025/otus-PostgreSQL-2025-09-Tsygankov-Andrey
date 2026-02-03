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

Узнаю PID = 11083 сессии чёрного окна, создаю таблицу test_locks с текстовым полем, заполняю её случайными сгенерированными данными в размере 1 миллион строк и запускаю UPDATE этого текстового поля по всей таблице
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
<img width="1657" height="671" alt="image" src="https://github.com/user-attachments/assets/e532cbb2-a8cf-4a73-ba95-62be775d6a70" /><br>

Параллельно во второй сессии PID = 11861 синего окна тоже запускаю UPDATE текстового поля по всей таблице test_locks, второй UPDATE ожидает завершения первого UPDATE и только после этого начинает выполняться
```
sudo -u postgres psql --cluster 18/otus

postgres=# SELECT pg_backend_pid();
postgres=# UPDATE test_locks set Data = Data || '+test2';
```
<img width="1065" height="491" alt="image" src="https://github.com/user-attachments/assets/29112083-f73c-4c19-ac0f-a7f186f326d7" /><br>




<img width="1657" height="671" alt="image" src="https://github.com/user-attachments/assets/e6ba9d39-a3d1-4b20-8f79-ffb10fe0803e" />

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
