## Подключение Oracle 12.1 к PostgreSQL через DBLink

### Установка ODBC-драйвера на сервере БД PostgreSQL
Первым шагом на всякий случай удаляем уже установленные библиотеки (если были установлены более старые версии – до 13). Добавляем репозиторий PostgreSQL и устанавливаем пакет
```
yum remove postgresql-odbc
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install -y postgresql13-odbc
```
<img width="1305" height="1380" alt="image" src="https://github.com/user-attachments/assets/efa6a332-884f-40f1-9e5e-8da33fba97d2" />
<img width="1305" height="1380" alt="image" src="https://github.com/user-attachments/assets/f75bd978-e70c-4c0b-ad9f-8a57f57e9cbd" />
<img width="1305" height="1380" alt="image" src="https://github.com/user-attachments/assets/d68c7c03-5be1-4331-a230-02bde91c169b" />
<img width="1305" height="1380" alt="image" src="https://github.com/user-attachments/assets/ae5e933c-80b0-483a-b866-3f06f7304c0c" />
<img width="1305" height="1271" alt="image" src="https://github.com/user-attachments/assets/e89d0bdf-859a-410b-9ed6-84fd242f26b2" /><br>

### Настройка odbcinst.ini

В файле `/etc/odbcinst.ini` появилась конфигурация для PostgreSQL, поправить путь для файла `/usr/pgsql-13/lib/psqlodbcw.so`
```
[PostgreSQL]
Description	= ODBC for PostgreSQL
Driver      = /usr/pgsql-13/lib/psqlodbcw.so
Setup       = /usr/lib/libodbcpsqlS.so
Driver64    = /usr/pgsql-13/lib/psqlodbcw.so
Setup64     = /usr/lib64/libodbcpsqlS.so
FileUsage   = 1

```

### Настройка odbc.ini
Создаю файл `/etc/odbc.ini` и добавляю в него параметры подключения подключения непосредственно к базе данных PostgreSQL:
```
[pgodbc]
Driver = PostgreSQL
Servername = 192.168.101.55
Port = 5432
Database = S22ATK.PGS.TST.RU
SSLmode = disable
```

Проверка соединениния через ODBC
```
isql -v pgodbc tst pgs_2020
```
И выборка из произвольной таблицы - соединение работает
```
SQL> select * from asuds.ns_prgr;
```

<img width="1305" height="941" alt="image" src="https://github.com/user-attachments/assets/f8b786fa-8f77-4457-b100-6720fa65e2a9" />

### Настройка Oracle Gateway на сервере БД Oracle (initpgodbc.ora)
Файл с параметрами инициализации подключения из Oracle. Расположение: $ORACLE_HOME/hs/admin/. Имя файла должно иметь вид init<sid>.ora, где <sid> – имя DSN для ODBC с учётом регистра (в нашем случае pgodbc)

