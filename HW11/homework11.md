# Домашнее задание 11
## Секционирование таблицы

Для выполнения домашнего задания разворачиваю [демонстрационную базу данных](https://postgrespro.ru/education/demodb) для СУБД PostgreSQL, которая в качестве предметной области использует авиаперевозки, выбираю объём данных по полётам за год - https://edu.postgrespro.ru/demo-20250901-1y.sql.gz
```
gunzip -c /tmp/demo-20250901-1y.sql.gz | psql -U postgres --cluster 18/otus
```
<img width="1849" height="1241" alt="image" src="https://github.com/user-attachments/assets/10453d9f-b351-4969-8191-0a927606a5d5" />
...
<img width="1849" height="1241" alt="image" src="https://github.com/user-attachments/assets/90c85f06-547c-4a57-b67f-eef6aec6d331" /><br><br>

Проанализировав схему данных, объёмы данных в таблицах и скорость выполнения запросов, выбираю для секционирования таблицу бронирований `bookings`, так как она является основной сущностью схемы данных, а выборка по диапазону дат бронирования требует значительного времени и ресурсов базы данных
<img width="1572" height="1235" alt="image" src="https://github.com/user-attachments/assets/53f23af1-c79e-48d5-bfbe-db90c46110a0" />

Некоторые таблицы по объёму превосходят таблицу бронирований `bookings`, но выборки из них в логике схемы данных производятся по ключевым полям, выполняются быстро и не требуют значительных ресурсов

<img width="1054" height="348" alt="image" src="https://github.com/user-attachments/assets/19a13d33-eb11-4927-9ac0-dc5326b4696a" /><br>
