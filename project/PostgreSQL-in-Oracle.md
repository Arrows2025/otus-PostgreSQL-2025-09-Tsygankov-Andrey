## Подключение Oracle 11 к PostgreSQL через DBLink

### Установка ODBC-драйвера на сервере БД Oracle
Первым шагом на всякий случай удаляем уже установленные библиотеки (если были установлены более старые версии – до 13). Добавляем репозиторий PostgreSQL и устанавливаем пакет
```
yum remove postgresql-odbc
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install -y postgresql13-odbc
```
<img width="2560" height="1380" alt="image" src="https://github.com/user-attachments/assets/f27ca5a5-5066-4407-9214-98aeb27dbf7e" />
<img width="2552" height="311" alt="image" src="https://github.com/user-attachments/assets/feb8f1f2-944e-487c-ab11-f4f622deca26" /><br>

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

Проверка соединения через ODBC
```
isql -v pgodbc tst pgs_2020
```
И выборка из произвольной таблицы - соединение работает
```
SQL> select * from asuds.ns_prgr;
```

<img width="2153" height="701" alt="image" src="https://github.com/user-attachments/assets/d4a43639-f4d4-4e35-9bb7-184cc79161ba" /><br>


### Настройка Oracle Gateway (initpgodbc.ora)
Файл с параметрами инициализации подключения из Oracle. Расположение: $ORACLE_HOME/hs/admin/. Имя файла должно иметь вид init<sid>.ora, где <sid> – имя DSN для ODBC с учётом регистра (в нашем случае pgodbc)

$ORACLE_HOME = '/u01/oracle/ora11'

```
HS_FDS_CONNECT_INFO = pgodbc
HS_FDS_SHAREABLE_NAME = /usr/pgsql-13/lib/psqlodbcw.so
HS_LANGUAGE = AMERICAN_AMERICA.AL32UTF8
HS_FDS_TRACE_LEVEL = OFF

set ODBCINI=/etc/odbc.ini
set ODBCSYSINI=/etc
```

### Настройка listener.ora
Добавляем настройки в прослушиватель для обработки входящих запросов на подключение. Расположение: $ORACLE_HOME/network/admin/
```
(SID_DESC =
  (SID_NAME = pgodbc)
  (PROGRAM = dg4odbc)
  (ENVS = "LD_LIBRARY_PATH=/usr/lib64:/usr/pgsql-13/lib")
  (ORACLE_HOME = /u01/oracle/ora11)
)
```
После обновления конфигурации перезапускаем прослушиватель `lsnrctl reload`
<img width="1417" height="431" alt="image" src="https://github.com/user-attachments/assets/1632a332-30a4-444f-a5a8-3ad5c650cc77" /><br>

### Настройка tnsnames.ora
И добавляем описание имени подключения. Расположение: $ORACLE_HOME/network/admin/.
```
PGODBC.ASUST.TST.RZD =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.101.50)(PORT = 1521))
    (CONNECT_DATA = (SID = pgodbc))
    (HS = OK)
  )
```
Проверка подключения `tnsping PGODBC`

<img width="2329" height="461" alt="image" src="https://github.com/user-attachments/assets/2a8e316b-f7c4-49b6-a9d5-c408676e1470" />




