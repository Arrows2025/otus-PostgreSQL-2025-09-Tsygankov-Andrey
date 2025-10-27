## Подключение Oracle 11 к PostgreSQL через DBLink

### Установка ODBC-драйвера на сервере БД Oracle
Устанавливаю пакет `postgresql-odbc`
```
yum install postgresql-odbc
```
<img width="2560" height="1380" alt="image" src="https://github.com/user-attachments/assets/f27ca5a5-5066-4407-9214-98aeb27dbf7e" />
<img width="2552" height="311" alt="image" src="https://github.com/user-attachments/assets/feb8f1f2-944e-487c-ab11-f4f622deca26" /><br>

### Настройка odbcinst.ini

В файле `/etc/odbcinst.ini` появилась конфигурация для PostgreSQL, поправить путь для файла `/usr/pgsql-15/lib/psqlodbcw.so`
```
[PostgreSQL]
Description	= ODBC for PostgreSQL (Unicode)
Driver      = /usr/pgsql-15/lib/psqlodbcw.so
Setup       = /usr/lib64/libodbcpsqlS.so
Driver64    = /usr/pgsql-15/lib/psqlodbcw.so
Setup64     = /usr/lib64/libodbcpsqlS.so
FileUsage   = 1

```

### Настройка odbc.ini
Создаю файл `/etc/odbc.ini` и добавляю в него параметры подключения подключения непосредственно к базе данных PostgreSQL:
```
[PG]
Description = PG
Driver = /usr/lib64/psqlodbc.so
ServerName = 192.168.101.55
Username = tst
Password = pgs_2020
Port = 5432
Database = S22ATK.PGS.TST.RU
[Default]
Driver = /usr/lib64/liboplodbcS.so
```

Проверка соединения через ODBC
```
isql -v PG tst pgs_2020
```
И выборка из произвольной таблицы - соединение работает
```
SQL> select * from asuds.ns_prgr;
```
<img width="2153" height="701" alt="image" src="https://github.com/user-attachments/assets/d4a43639-f4d4-4e35-9bb7-184cc79161ba" /><br>


### Настройка Oracle Gateway (initPG.ora)
Файл с параметрами инициализации подключения из Oracle. Расположение: $ORACLE_HOME/hs/admin/. Имя файла должно иметь вид initSID.ora, где SID – имя DSN для ODBC с учётом регистра (в нашем случае PG)

$ORACLE_HOME = '/u01/oracle/ora11'

```
# This is a sample agent init file that contains the HS parameters that are
# needed for the Database Gateway for ODBC

#
# HS init parameters
#
HS_FDS_CONNECT_INFO = PG
HS_FDS_TRACE_LEVEL = 0
HS_FDS_SHAREABLE_NAME = /usr/lib64/psqlodbc.so
HS_LANGUAGE = AMERICAN_AMERICA.AL32UTF8
#
# ODBC specific environment variables
#
set ODBCINI=/etc/odbc.ini
```
Кодировка AMERICAN_AMERICA.WE8ISO8859P9 неверно передаёт русские буквы, с кодировкой `AMERICAN_AMERICA.AL32UTF8` не коннектится DB Link

### Настройка listener.ora
Добавляем настройки в прослушиватель для обработки входящих запросов на подключение. Расположение: $ORACLE_HOME/network/admin/
```
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME=PG)
      (ORACLE_HOME=/u01/oracle/ora11)
      (PROGRAM=dg4odbc)
    )
  )
```
После обновления конфигурации перезапускаем прослушиватель `lsnrctl reload` или `lsnrctl stop` и `lsnrctl start`
<img width="1417" height="431" alt="image" src="https://github.com/user-attachments/assets/1632a332-30a4-444f-a5a8-3ad5c650cc77" /><br>

### Настройка tnsnames.ora
И добавляем описание имени подключения. Расположение: $ORACLE_HOME/network/admin/.
```
PG.ASUST.TST.RZD =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.101.50)(PORT = 1521))
    (CONNECT_DATA =
    (SID = PG)
  )
  (HS = OK)
)
```
Проверка подключения `tnsping PG.ASUST.TST.RZD`
<img width="2265" height="461" alt="image" src="https://github.com/user-attachments/assets/89379499-e114-4da3-b246-cc8e7038099a" />





