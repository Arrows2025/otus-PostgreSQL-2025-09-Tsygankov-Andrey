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

Отключаю автоматический запуск сервиса PostgreSQL `sudo systemctl disable postgresql.service`
<img width="1801" height="221" alt="image" src="https://github.com/user-attachments/assets/1c923260-e1c5-46af-bbb6-9df4e35869da" /><br>

Устанавливаю etcd
```
sudo apt-get update                  # обновление системы
sudo apt-get install etcd-server     # установка etcd-server
sudo apt-get install etcd-client     # установка etcd-client
etcd --version                       # проверка установки
```
<img width="2537" height="641" alt="image" src="https://github.com/user-attachments/assets/e083d8bd-7fba-424e-9b06-9ba2d0293149" />
<img width="2537" height="1361" alt="image" src="https://github.com/user-attachments/assets/664912f2-e361-4080-bb49-db7a219b3d48" />
<img width="2537" height="1031" alt="image" src="https://github.com/user-attachments/assets/b7553652-3db2-401f-be5f-a343db2a7c6d" />
<img width="2537" height="221" alt="image" src="https://github.com/user-attachments/assets/491ca40e-1520-4799-bb42-dfb63cea3c67" /><br><br>

Удаляю конфигурацию etcd по умолчанию
```
sudo systemctl stop etcd
sudo systemctl disable etcd
sudo rm -rf /var/lib/etcd/default
```
<img width="1705" height="281" alt="image" src="https://github.com/user-attachments/assets/eddc7e25-5f64-4c52-9bb0-ab53c9afad1b" /><br>





Проверяю статус etcd `systemctl status etcd`
<img width="2361" height="761" alt="image" src="https://github.com/user-attachments/assets/d8e552eb-f9ad-4682-a3a5-33b1daccb7a8" /><br>

Настраиваю etcd на трёх узлах `sudo nano /etc/default/etcd`

```
ETCD_NAME="etcd-node-0"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.50:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.50:2379"
ETCD_INITIAL_CLUSTER="etcd-node-0=http://192.168.0.50:2380,etcd-node-1=http://192.168.0.51:2380,etcd-node-2=http://192.168.0.52:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
ETCD_DATA_DIR="/var/lib/etcd/pg-cluster"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"
```

```
ETCD_NAME="etcd-node-1"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.51:2380"
ETCD_INITIAL_CLUSTER="etcd-node-0=http://192.168.0.50:2380,etcd-node-1=http://192.168.0.51:2380,etcd-node-2=http://192.168.0.52:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.51:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
ETCD_DATA_DIR="/var/lib/etcd/pg-cluster"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"
```

```
ETCD_NAME="etcd-node-2"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.0.52:2380"
ETCD_INITIAL_CLUSTER="etcd-node-0=http://192.168.0.50:2380,etcd-node-1=http://192.168.0.51:2380,etcd-node-2=http://192.168.0.52:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.52:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="pg-cluster"
ETCD_DATA_DIR="/var/lib/etcd/pg-cluster"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"
```

Запускаю etcd на трёх узлах `sudo systemctl start etcd' и проверяю состояние кластера etcd, кластер стартован успешно
```
etcdctl member list
etcdctl endpoint --cluster health
etcdctl endpoint status --cluster -w table
```
<img width="2393" height="551" alt="image" src="https://github.com/user-attachments/assets/21eea86f-67d2-46b8-b1ed-d0e97d42ea81" /><br>

Останавливаю etcd для корректировки параметра ETCD_INITIAL_CLUSTER_STATE="new" на ETCD_INITIAL_CLUSTER_STATE="existing" и вновь запускаю кластер
