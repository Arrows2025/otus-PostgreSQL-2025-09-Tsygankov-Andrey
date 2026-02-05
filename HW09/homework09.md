# Домашнее задание 09
## Работа с join'ами, статистикой

Для выполнения домашнего задания в новой схеме `movies` создан набор таблиц для задачи "Мини-фильмотека", таблицы заполнены некоторыми данными
```sql
CREATE SCHEMA movies;
-- Список фильмов с атрибутами
CREATE TABLE movies.movie (
    id_movie SERIAL PRIMARY KEY,
    name_movie varchar(50),
    year integer,
);
-- Список жанров
CREATE TABLE movies.genre (
    id_genre SERIAL PRIMARY KEY,
    name_genre varchar(20)
);
-- Таблица отношений фильмов и жанров
CREATE TABLE movies.genres (
    id_movie integer,
    id_genre integer
);
-- Список стран
CREATE TABLE movies.country (
    id_country SERIAL PRIMARY KEY,
    name_country varchar(20)
);
-- Таблица отношений фильмов и стран
CREATE TABLE movies.countries (
    id_movie integer,
    id_country integer
);
-- Список пользователей
CREATE TABLE movies.users (
    id_user SERIAL PRIMARY KEY,
    name_user varchar(20)
);
-- Таблица с пользовательскими оценками по каждому фильму
CREATE TABLE movies.values (
    id_movie integer,
    id_user integer,
    value integer
);
-- Заполненные данные
INSERT INTO movies.movie (name_movie,"year") VALUES
	 ('Интерстеллар',2014),
	 ('Побег из Шоушенка',1994),
	 ('Зелёная миля',1999),
	 ('Форрест Гамп',1994),
	 ('Брат',1997),
	 ('Собачье сердце',1988);

INSERT INTO movies.genre (name_genre) VALUES
	 ('фантастика'),
	 ('драма'),
	 ('приключения'),
	 ('фэнтези'),
	 ('криминал'),
	 ('комедия'),
	 ('мелодрама'),
	 ('история'),
	 ('военный'),
	 ('боевик');
INSERT INTO movies.genre (name_genre) VALUES
	 ('мультфильм'),
	 ('ужасы'),
	 ('триллер');

INSERT INTO movies.genres (id_movie,id_genre) VALUES
	 (1,1),
	 (1,2),
	 (1,3),
	 (2,2),
	 (3,2),
	 (3,4),
	 (3,5),
	 (4,2),
	 (4,6),
	 (4,7);
INSERT INTO movies.genres (id_movie,id_genre) VALUES
	 (4,8),
	 (4,9),
	 (5,2),
	 (5,5),
	 (5,10);

INSERT INTO movies.country (name_country) VALUES
	 ('США'),
	 ('Великобритания'),
	 ('Канада'),
	 ('Россия'),
	 ('Франция'),
	 ('Япония');

INSERT INTO movies.countries (id_movie,id_country) VALUES
	 (1,1),
	 (1,2),
	 (1,3),
	 (2,1),
	 (3,1),
	 (4,1),
	 (5,4);

INSERT INTO movies.users (name_user) VALUES
	 ('Пользователь 1'),
	 ('Пользователь 2'),
	 ('Пользователь 3'),
	 ('Пользователь 4'),
	 ('Пользователь 5'),
	 ('Пользователь 6'),
	 ('Пользователь 7'),
	 ('Пользователь 8'),
	 ('Пользователь 9'),
	 ('Пользователь 10');

INSERT INTO movies."values" (id_movie,id_user,value) VALUES
	 (1,1,7),
	 (1,2,8),
	 (1,5,7),
	 (1,6,8),
	 (1,7,9),
	 (1,8,8),
	 (1,10,10),
	 (2,1,8),
	 (2,2,10),
	 (2,3,7);
INSERT INTO movies."values" (id_movie,id_user,value) VALUES
	 (2,4,10),
	 (2,5,10),
	 (2,6,9),
	 (2,7,9),
	 (2,8,10),
	 (2,9,8),
	 (2,10,9),
	 (3,4,9),
	 (3,1,10),
	 (3,7,8);
INSERT INTO movies."values" (id_movie,id_user,value) VALUES
	 (3,9,9),
	 (4,2,7),
	 (4,3,8),
	 (4,4,9),
	 (4,6,7),
	 (4,7,9),
	 (4,8,8),
	 (4,9,7),
	 (4,10,6),
	 (5,1,6);
INSERT INTO movies."values" (id_movie,id_user,value) VALUES
	 (5,3,9),
	 (5,4,7),
	 (5,5,9),
	 (5,7,7),
	 (5,8,9),
	 (5,9,9),
	 (5,10,8),
	 (5,2,7),
	 (3,3,10);
```

1️⃣ Прямое соединение (INNER JOIN)
```sql
SELECT *
FROM movies.movie m
INNER JOIN movies.genres mg ON m.id_movie = mg.id_movie
INNER JOIN movies.genre g on g.id_genre = mg.id_genre;
```
Выборка показывает список фильмов и полный список всех жанров ко всем фильмам, у которых существует хотя бы один жанр в таблице `genres`
<img width="1178" height="644" alt="image" src="https://github.com/user-attachments/assets/edfe011c-4d9d-4e56-8603-f3def5565abf" /><br>

2️⃣ Левостороннее соединение (LEFT JOIN)
```sql
SELECT *
FROM movies.movie m
LEFT JOIN movies.genres mg ON m.id_movie = mg.id_movie
LEFT JOIN movies.genre g on g.id_genre = mg.id_genre;
```
Выборка показывает список всех фильмов, как с существующим жанром, так и без него
<img width="1175" height="668" alt="image" src="https://github.com/user-attachments/assets/c24f75c6-415b-455a-adaf-65c69b130bc6" /><br>

2️⃣ Левостороннее соединение (LEFT JOIN)
```sql
SELECT *
FROM movies.movie m
LEFT JOIN movies.genres mg ON m.id_movie = mg.id_movie
LEFT JOIN movies.genre g on g.id_genre = mg.id_genre;
```
Выборка показывает список всех фильмов, как с существующим жанром, так и без него
<img width="1175" height="668" alt="image" src="https://github.com/user-attachments/assets/c24f75c6-415b-455a-adaf-65c69b130bc6" /><br>





