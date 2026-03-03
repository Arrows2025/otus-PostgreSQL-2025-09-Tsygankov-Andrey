# Домашнее задание 14
## Репликация

Для выполнения домашнего используются три ранее установленные виртуальные машины с процессором на 1 ядро и 2 Гб оперативной памяти, на каждую виртуальную машину установлена серверная операционная система Ubuntu 24.04.3:
Ubuntu24Server - IP 192.168.0.50
Ubuntu24Node1 - IP 192.168.0.51
Ubuntu24Node2 - IP 192.168.0.52
<img width="1089" height="624" alt="image" src="https://github.com/user-attachments/assets/4181101a-49b5-4d3e-a5e3-50335b5bb097" /><br>

Подключаюсь к базе данных на ВМ1 и устанавливаю `wal_level = logical` для логической репликации
```
sudo -u postgres psql --cluster 18/otus

postgres=# ALTER SYSTEM SET wal_level = logical;
ALTER SYSTEM
postgres=# \q
```
Перегружаю кластер и заново подключаюсь, создаю базу данных `replica`, в ней две таблицы `test` и `test2`, на таблицу `test` создаю публикацию `test_pub`
```
sudo pg_ctlcluster 18 otus restart
sudo -u postgres psql --cluster 18/otus

postgres=# CREATE DATABASE replica;
CREATE DATABASE
postgres=# \c replica;
Вы подключены к базе данных "replica" как пользователь "postgres".
replica=# CREATE TABLE test (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE
replica=# CREATE TABLE test2 (id SERIAL PRIMARY KEY, Data TEXT);
CREATE TABLE
replica=# CREATE PUBLICATION test_pub FOR TABLE test;
CREATE PUBLICATION
```
<img width="1097" height="731" alt="image" src="https://github.com/user-attachments/assets/9bfbb29f-588b-4406-a5aa-0b3eecc43044" /><br>

На ВМ2 создаю новый кластре otus2, настраиваю      устанавливаю `wal_level = logical` для логической репликации
