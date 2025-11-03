# Домашнее задание 03
## Установка и настройка PostgreSQL

Для выполнения домашнего задания установлен сервер Ubuntu 24.04.3 на виртуальной машине, используя Oracle VirtualBox Manager. Подключаюсь к серверу через программу PuTTY посредством ввода своего пользователя с авторизацией с помощью ключа

<img width="1305" height="1061" alt="image" src="https://github.com/user-attachments/assets/0d40a3d3-4e12-4cd8-9673-f190177fc8d3" /><br>

Устанавливаю PostgreSQL 18 и набор пакетов дополнительных программ
```sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql && sudo apt install unzip && sudo apt -y install mc```

Проверка статуса кластера PostgreSQL: `pg_lsclusters`

<img width="1305" height="941" alt="image" src="https://github.com/user-attachments/assets/aa296430-3964-4375-ab8c-0e8a61b5831e" /><br>

Захожу в БД PostgreSQL под пользователем postgres `sudo -u postgres psql` и создаю таблицу `ns_prgr` с небольшим содержимым
```
postgres=# CREATE TABLE ns_prgr (
        prgr int2 NOT NULL,
        nam_prgr bpchar(8) NULL,
        namprgrs bpchar(3) NULL,
        grup numeric NULL,
        tip int2 DEFAULT 0 NOT NULL,
        npp_graf numeric NULL,
        cod_color numeric NULL,
        CONSTRAINT xpkds007 PRIMARY KEY (prgr)
);
CREATE TABLE
postgres=# INSERT INTO ns_prgr (prgr, nam_prgr, namprgrs, grup, tip, npp_graf, cod_color) VALUES(0, 'Всего   ', 'Всг', NULL, 1, NULL, NULL);
INSERT INTO ns_prgr (prgr, nam_prgr, namprgrs, grup, tip, npp_graf, cod_color) VALUES(1, 'Порожний', 'Пор', 0, 1, NULL, NULL);
INSERT INTO ns_prgr (prgr, nam_prgr, namprgrs, grup, tip, npp_graf, cod_color) VALUES(2, 'Груженый', 'Гр.', 0, 1, NULL, NULL);
INSERT INTO ns_prgr (prgr, nam_prgr, namprgrs, grup, tip, npp_graf, cod_color) VALUES(3, 'Перегруз', 'Пгр', 2, 0, NULL, NULL);
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
postgres=# \q
```

<img width="2281" height="791" alt="image" src="https://github.com/user-attachments/assets/0c7f59ac-4fad-4e77-9bc6-09fc3a0f73d1" /><br>

Проверяю наличие дисков в системе `lsblk`
<img width="1033" height="311" alt="image" src="https://github.com/user-attachments/assets/aeba3b11-8892-4e7d-8cbf-3cf0f4aeb6fe" /><br>

Останавливаю виртуальную машину и с помощью панели управления VirtualBox добавляю новый виртуальный диск объёмом 10 Gb
<img width="777" height="454" alt="image" src="https://github.com/user-attachments/assets/0f40fb2c-571a-4761-b4cb-d1e0a8e6507b" /><br>

<img width="1203" height="504" alt="image" src="https://github.com/user-attachments/assets/05811dda-2ac2-49fd-9401-adcc7203282a" /><br>

Запускаю виртуальную машину и проверяю наличие дисков в системе `lsblk`, в системе появился новый неразмеченный диск `sdb`
<img width="1033" height="341" alt="image" src="https://github.com/user-attachments/assets/3213fd2d-2206-4170-8991-baf349e71a6f" /><br>

Размечаю диск, добавляю раздел, форматирую его под файловую систему `ext4` и монтирую диск в систему
```
sudo parted /dev/sdb mklabel gpt -- создаю таблицу разделов для диска sdb
sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100% -- создаю раздел на диске sdb
sudo mkfs.ext4 -L new-data /dev/sdb1 - создаю файловую систему для раздела
sudo mkdir -p /mnt/new-data -- создаю директорию для монтирования файловой системы
sudo mount -o defaults /dev/sdb1 /mnt/new-data
df -h -- проверка примонтированных устройств в системе, новый раздел диска примонтирован по пути /mnt/new-data
```
<img width="1321" height="971" alt="image" src="https://github.com/user-attachments/assets/5ad07dbe-d276-486b-aa5c-21c18f5e31c7" /><br>

Для автоматической загрузки нового раздела нахожу идентификатор диска командой `blkid` и добавляю строку загрузки раздела в файл `/etc/fstab`
```
UUID="f5b15ab5-3166-41dc-a21e-3fd9635b6571" /mnt/new-data ext4 defaults 0 2
```
<img width="2745" height="221" alt="image" src="https://github.com/user-attachments/assets/23cd5d5c-9589-429b-87c9-f24492b7319b" /><br>

Для этого делаю бекап файла `/etc/fstab` и редактирую его с помощью редактора `nano`
```
sudo cp /etc/fstab /etc/fstab.bak
sudo nano /etc/fstab
```
После перезапуска операционной системы новый диск монтируется в систему автоматически
<img width="1305" height="1271" alt="image" src="https://github.com/user-attachments/assets/58223fe3-4930-4483-afc3-2829d9cab09b" /><br>

Проверяю статус кластера `pg_lsclusters` и останавливаю PostgreSQL `sudo systemctl stop postgresql@18-main`
<img width="1689" height="281" alt="image" src="https://github.com/user-attachments/assets/d0b7908c-3dd8-4f6a-a436-b5bf59e982e3" /><br>

Делаю пользователя `postgres` владельцем `/mnt/new-data`, переношу данные БД PostgreSQL на новый диск в директорию `/mnt/new-data`, пробую запустить PostgreSQL
```
sudo chown -R postgres:postgres /mnt/new-data
sudo mv /var/lib/postgresql/18 /mnt/new-data
sudo systemctl start postgresql@18-main
pg_lsclusters
```
<img width="1705" height="251" alt="image" src="https://github.com/user-attachments/assets/90da3a2f-74bf-4960-b0dc-42a5ffb3ce98" />

PostgreSQL не запустился из-за того, что не смог обнаружить папку со базой данных

Меняю параметр каталога, в котором хранятся файлы баз данных и конфигурация кластера `data_directory` в файле настроек PostgreSQL `/etc/postgresql/18/main/postgresql.conf` на директорию, в которой сейчас находятся файлы базы данных PostgreSQL `data_directory = '/mnt/new-data/18/main'`, пробую запустить PostgreSQL. PostgreSQL стартует, так как настройки сооствествуют местонахождению каталога баз данных PostgreSQL. Захожу в БД PostgreSQL под пользователем postgres `sudo -u postgres psql` и проверяю наличие таблицы с данными `ns_prgr`. Таблица на месте, нужные данные подключены к PostgreSQL.
```
sudo nano /etc/postgresql/18/main/postgresql.conf
sudo systemctl start postgresql@18-main
pg_lsclusters
sudo -u postgres psql
postgres=# select * from ns_prgr;
postgres-# \q
```
<img width="1593" height="668" alt="image" src="https://github.com/user-attachments/assets/3a88d863-78ce-46ca-a061-ea3af8d87ce4" /><br>
