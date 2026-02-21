# Домашнее задание 11
## Секционирование таблицы

Для выполнения домашнего задания разворачиваю https://postgrespro.ru/education/demodb демонстрационную базу данных для СУБД PostgreSQL, которая в качестве предметной области использует авиаперевозки, выбираю объём данных по полётам за год - https://edu.postgrespro.ru/demo-20250901-1y.sql.gz  
создаю и заполняю таблицу "Справочник грузов" `nsgruz`. Структура таблицы с данными представлена ниже
```
gunzip -c /tmp/demo-20250901-1y.sql.gz | psql -U postgres --cluster 18/otus
```


<img width="1849" height="1241" alt="image" src="https://github.com/user-attachments/assets/10453d9f-b351-4969-8191-0a927606a5d5" />
...
<img width="1849" height="1241" alt="image" src="https://github.com/user-attachments/assets/90c85f06-547c-4a57-b67f-eef6aec6d331" />


