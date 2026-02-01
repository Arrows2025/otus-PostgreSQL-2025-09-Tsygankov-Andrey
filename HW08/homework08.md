# Домашнее задание 08
## Механизм блокировок

1️⃣ Настраиваю сервер, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Для этого проверяю и включаю параметр `log_lock_waits`, устанавливаю параметр `deadlock_timeout` равным 200 мс и перезагружаю файлы конфигурации PostgreSQL
```
sudo -u postgres psql

postgres=# show log_lock_waits;
postgres=# ALTER SYSTEM SET log_lock_waits = on;
postgres=# show deadlock_timeout;
postgres=# ALTER SYSTEM SET deadlock_timeout = 200;
postgres=# SELECT pg_reload_conf();
postgres=# show log_lock_waits;
postgres=# show deadlock_timeout;
```
<img width="857" height="1211" alt="image" src="https://github.com/user-attachments/assets/3d99012f-46a2-42cf-92bc-9902e73fceb1" /><br>



:two: Для инициализации Pgbench создаю для тестов базу данных pbtest и инициализирую утилиту Pgbench

