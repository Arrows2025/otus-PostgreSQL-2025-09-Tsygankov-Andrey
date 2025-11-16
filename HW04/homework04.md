# Домашнее задание 04
## Работа с базами данных, пользователями и правами

В базе данных PostgreSQL 18 проверяю наличие кластеров `pg_lsclusters`
```diff
+18  main    5432 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
```
Cоздаю и сразу стартую новый кластер `sudo pg_createcluster 18 otus --start`
```diff
+18  otus    5433 online postgres /var/lib/postgresql/18/otus /var/log/postgresql/postgresql-18-otus.log
```
В списке кластеров базы данных PostgreSQL 18 появляется новый кластер `otus`
```diff
+18  main    5432 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
+18  otus    5433 online postgres /var/lib/postgresql/18/otus /var/log/postgresql/postgresql-18-otus.log
```

<img width="1689" height="1091" alt="image" src="https://github.com/user-attachments/assets/0495641c-d212-4492-a09b-d2ebe99c0425" /><br>

Для настройки удалённого доступа к новому кластеру редактирую конфигурационный файл PostgreSQL `sudo nano /etc/postgresql/18/otus/postgresql.conf` для прослушивания всех адресов

`#listen_addresses = 'localhost'          # what IP address(es) to listen on;`  
меняю на  
`listen_addresses = '*'                   # what IP address(es) to listen on;`

И редактирую конфигурационный файл PostgreSQL: `sudo nano /etc/postgresql/18/otus/pg_hba.conf` для доступа к БД PostgreSQL с моей локальной машины

добавляю в конец файла строку
`host    all             all             192.168.0.129/24            scram-sha-256`

Меняю пароль для пользователя базы данных `postgres`
```
sudo -u postgres psql --cluster 18/otus
\password
\q
```

Перезагрузка кластера базы данных `sudo pg_ctlcluster 18 otus restart` и тест соединения с БД PostgreSQL через приложение DBeaver прошёл успешно

<img width="893" height="775" alt="image" src="https://github.com/user-attachments/assets/10b9d991-ed8b-4eed-976b-45d10cf8dede" /><br>

