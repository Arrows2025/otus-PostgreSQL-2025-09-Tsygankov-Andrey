# Домашнее задание 08
## Механизм блокировок

1️⃣ Настраиваю кластер, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Для этого проверяю и включаю параметр `log_lock_waits`, устанавливаю параметр `deadlock_timeout` равным 200 мс и перезагружаю файлы конфигурации PostgreSQL
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

В журнале появилась запись о блокировке, как только время ожидания превысило 200 мс `Process holding the lock: 11083. Wait queue: 11861.`
<img width="2521" height="521" alt="image" src="https://github.com/user-attachments/assets/17d64905-fca2-4420-b333-4a008f0c6631" /><br>

2️⃣ Открываю три сессии коннекта к базе данных в разных окнах, в каждом окне проверяю PID сессии и по очереди запускаю UPDATE одной и той же строки

```
sudo -u postgres psql --cluster 18/otus

postgres=# SELECT pg_backend_pid();
postgres=# begin;
postgres=*# UPDATE test_locks set data = 'Session 1' where id = 1;
```
Первая сессия PID = 52652 синее окно
<img width="1097" height="491" alt="image" src="https://github.com/user-attachments/assets/0deed66d-04e4-4ba1-939d-9bea5ad0b7cd" /><br>
Вторая сессия PID = 52766 зелёное окно
<img width="1097" height="461" alt="image" src="https://github.com/user-attachments/assets/04b51de7-c2e3-48bb-b550-2bf051d7ee4b" /><br>
Третья сессия PID = 52831 красное окно
<img width="1097" height="461" alt="image" src="https://github.com/user-attachments/assets/dcbd56de-68cc-4a97-882b-1668b55552e1" />

В отдельном окне после запуска каждого UPDATE в разных сессиях проверяю наличие блокировок в pg_locks
```
postgres=# SELECT locktype, relation::REGCLASS, mode, granted, pid, pg_blocking_pids(pid) AS wait_for
FROM pg_locks WHERE relation = 'test_locks'::regclass order by pid;
```
<img width="1657" height="1181" alt="image" src="https://github.com/user-attachments/assets/4632a97d-cad6-4ddb-b32a-00c0575b540d" /><br>






3️⃣

4️⃣
<img width="1305" height="461" alt="image" src="https://github.com/user-attachments/assets/4cba337a-802d-4ee9-b525-ee71e8cc6afb" />
