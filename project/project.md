# Проектная работа
## Создание и тестирование высоконагруженного отказоустойчивого кластера PostgreSQL с использованием Patroni, etcd, HAProxy

Для проектной работы в Oracle VirtualBox Manager созданы три виртуальные машины с процессором на 1 ядро и 2 Гб оперативной памяти, на каждую виртуальную машину установлена серверная операционная система Ubuntu 24.04.3:
* Ubuntu24Server - IP 192.168.0.50
* Ubuntu24Node1 - IP 192.168.0.51
* Ubuntu24Node2 - IP 192.168.0.52

Устанавливаю на каждую виртуальную машину базу данных PostgreSQL 18 с базовыми параметрами и пакетом дополнительных программ: `sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql && sudo apt install unzip && sudo apt -y install mc`

Редактирую конфигурационный файл PostgreSQL `sudo nano /etc/postgresql/18/main/postgresql.conf` для прослушивания всех адресов
```
#listen_addresses = 'localhost'          # what IP address(es) to listen on;
изменить на
listen_addresses = '*'                   # what IP address(es) to listen on;
```
Редактирую конфигурационный файл PostgreSQL `sudo nano /etc/postgresql/18/main/pg_hba.conf` для доступа к БД PostgreSQL с моей локальной машины
```
добавить в конец файла строку
host    all             all             192.168.0.129/24        scram-sha-256
```
Устанавливаю пароль для пользователя базы данных postgres
```
sudo -u postgres psql

postgres=# \password
postgres=# \q
```

Перезагрузка кластера базы данных `sudo pg_ctlcluster 18 main restart` и тест соединения с БД PostgreSQL через приложение DBeaver прошёл успешно

Устанавливаю etcd
```
sudo apt-get update                  # обновление системы
sudo apt-get install etcd-server     # установка etcd
etcd --version                       # проверка установки
```
<img width="2537" height="641" alt="image" src="https://github.com/user-attachments/assets/e083d8bd-7fba-424e-9b06-9ba2d0293149" />
<img width="2537" height="1361" alt="image" src="https://github.com/user-attachments/assets/664912f2-e361-4080-bb49-db7a219b3d48" />
<img width="2537" height="221" alt="image" src="https://github.com/user-attachments/assets/491ca40e-1520-4799-bb42-dfb63cea3c67" />


