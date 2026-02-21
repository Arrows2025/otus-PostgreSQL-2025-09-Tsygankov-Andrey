# Домашнее задание 11
## Секционирование таблицы

Для выполнения домашнего задания создаю и заполняю таблицу "Справочник грузов" `nsgruz`. Структура таблицы с данными представлена ниже
```sql
CREATE TABLE nsgruz (
	kod_gr bpchar(6) NOT NULL,
	kl_gr bpchar(1) NULL,
	name_gr bpchar(15) NULL,
	name_grn bpchar(18) NULL,
	name_grk bpchar(18) NULL,
	ns_gruz_s bpchar(6) NULL,
	kod_group bpchar(3) NULL,
	kod_gr_etsng bpchar(6) NULL,
	CONSTRAINT xpknsgruz PRIMARY KEY (kod_gr)
);
```


<img width="1193" height="281" alt="image" src="https://github.com/user-attachments/assets/0c2d3178-b67e-4157-a686-d39ec84d53cf" />
