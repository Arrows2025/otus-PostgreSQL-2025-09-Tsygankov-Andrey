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

После первого UPDATE первая сессия PID = 52652 заблокировала таблицу `test_locks` в режиме RowExclusiveLock, granted = true, всё доступно для выполнения.

После второго UPDATE вторая сессия PID = 52766 заблокировала таблицу `test_locks` в режиме RowExclusiveLock и версию строки в режиме ExclusiveLock, granted = true, всё доступно для выполнения, но вторая сессия PID = 52766 ожидает завершения первой сессии PID = 52652

После третьего UPDATE третья сессия PID = 52831 заблокировала таблицу `test_locks` в режиме RowExclusiveLock и версию строки в режиме ExclusiveLock, granted у блокировки таблицы = true, granted у блокировки версии строки = false означает, что процесс блокировку не получил и будет её ожидать, так как минимум один другой процесс удерживает блокировку той же строки в конфликтующем режиме, третья сессия PID = 52831 ожидает завершения второй сессии PID = 52766, которая в свою очередь ожидает завершения первой сессии PID = 52652

После `COMMIT;` первой сессии PID = 52652 выполнилась вторая сессия PID = 52766, в pg_locks остались две блокировки на таблицу в режиме RowExclusiveLock, блокировки на версию строки пропали, конфликтующего режима на одну строку больше нет, так как и в первой и во второй сессии UPDATE этой строки уже выполнен. После `COMMIT;` второй сессии PID = 52766 выполнилась третья сессия PID = 52766, после `COMMIT;` третьей сессии PID = 52831 все блокировки ушли
<img width="1657" height="1031" alt="image" src="https://github.com/user-attachments/assets/166e08c7-00c7-408d-857e-e0864c3bc288" /><br>

3️⃣ Чтобы воспроизвести взаимоблокировку трёх транзакций создаю три таблицы и заполняю их произвольными данными

```
CREATE TABLE Table_A(
  id SERIAL PRIMARY KEY,
  Data TEXT
);

CREATE TABLE Table_B(
  id SERIAL PRIMARY KEY,
  Data TEXT
);

CREATE TABLE Table_C(
  id SERIAL PRIMARY KEY,
  Data TEXT
);

INSERT INTO Table_A (Data) VALUES ('Тест 1'),('Тест 2'),('Тест 3');
INSERT INTO Table_B (Data) VALUES ('Тест 1'),('Тест 2'),('Тест 3');
INSERT INTO Table_C (Data) VALUES ('Тест 1'),('Тест 2'),('Тест 3');
```
<img width="1113" height="911" alt="image" src="https://github.com/user-attachments/assets/3a741495-b6c6-4066-846f-bcc6245b579f" /><br>

В первой сессии PID = 82993 синего окна выполняю UPDATE на строку таблицы Table_A, во второй сессии PID = 83011 зелёного окна выполняю UPDATE на строку таблицы Table_B, в третьей сессии PID = 83039 красного окна выполняю UPDATE на строку таблицы Table_C

Далее в первой сессии синего окна выполняю UPDATE на строку таблицы Table_B, в второй сессии зелёного окна выполняю UPDATE на строку таблицы Table_C, а в третьей сессии красного окна выполняю UPDATE на строку таблицы Table_A, и в третьей сессии получаю взаимоблокировку
```
ОШИБКА:  обнаружена взаимоблокировка
ПОДРОБНОСТИ:  Процесс 83039 ожидает в режиме ShareLock блокировку "транзакция 780"; заблокирован процессом 82993.
Процесс 82993 ожидает в режиме ShareLock блокировку "транзакция 781"; заблокирован процессом 83011.
Процесс 83011 ожидает в режиме ShareLock блокировку "транзакция 782"; заблокирован процессом 83039.
```

Далее во второй сессии зелёного окна выполняю UPDATE на строку таблицы Table_A, и во второй сессии получаю взаимоблокировку
```
ОШИБКА:  обнаружена взаимоблокировка
ПОДРОБНОСТИ:  Процесс 83011 ожидает в режиме ShareLock блокировку "транзакция 780"; заблокирован процессом 82993.
Процесс 82993 ожидает в режиме ShareLock блокировку "транзакция 781"; заблокирован процессом 83011.
```
<img width="1849" height="611" alt="image" src="https://github.com/user-attachments/assets/65295618-add8-4dbc-a78b-c31f5ef902a2" />
<img width="1865" height="731" alt="image" src="https://github.com/user-attachments/assets/092009a4-23d2-4f95-b026-b8249bff6fe2" />
<img width="1849" height="701" alt="image" src="https://github.com/user-attachments/assets/e8b7fed3-7b7f-485f-8018-cd50e1266898" /><br><br>

В журнале сообщений отобразилась информация о воспроизведённых взаимоблокировках и запросах, которые к этим взаимоблокировкам привели, таким образом по журналу сообщений можно найти взаимоблокировки и причины их появления постфактум `tail -n 50 /var/log/postgresql/postgresql-18-otus.log`
<img width="2560" height="1380" alt="image" src="https://github.com/user-attachments/assets/763caf67-6d0c-4fe8-9663-00097daa1028" /><br>



4️⃣ Задание со ⭐: Две транзакции, выполняющие единственную команду UPDATE без WHERE одной и той же таблицы, могут вызвать взаимоблокировку.

Для проверки в первой сессии PID = 102270 синего окна запускаю UPDATE без WHERE таблицы `test_locks` с прямым ORDER BY, а во второй сессии PID = 102363 зелёного окна запускаю UPDATE без WHERE этой же таблицы с обратным ORDER BY. Таблица содержит один миллион строк и в какой-то момент оба запущенных UPDATE вышли на блокировку одних и тех же строк в конфликтующем режиме, что вызвало взаимоблокировку
```
UPDATE test_locks set Data = Data || '+test3' WHERE id IN (SELECT id FROM test_locks ORDER BY id ASC);
```
<img width="1881" height="431" alt="image" src="https://github.com/user-attachments/assets/5d788f99-39f4-42d3-b069-a3851cec2043" />

```
UPDATE test_locks set Data = Data || '+test4' WHERE id IN (SELECT id FROM test_locks ORDER BY id DESC);
```
<img width="1881" height="551" alt="image" src="https://github.com/user-attachments/assets/13edef98-a32f-4396-98c1-b80882330327" />


