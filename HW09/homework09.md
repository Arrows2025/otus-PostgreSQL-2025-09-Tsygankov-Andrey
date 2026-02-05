# Домашнее задание 09
## Работа с join'ами, статистикой

Для выполнения домашнего задания в новой схеме `movies` создан набор таблиц для задачи "Мини-фильмотека", таблицы заполнены произвольными данными
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

3️⃣ Правостороннее соединение (RIGHT JOIN)
```sql
SELECT *
FROM movies.movie m
RIGHT JOIN movies.genres mg ON m.id_movie = mg.id_movie
RIGHT JOIN movies.genre g on g.id_genre = mg.id_genre;
```
Выборка показывает список всех жанров, даже если для этого жанра нет никакого фильма
<img width="1172" height="712" alt="image" src="https://github.com/user-attachments/assets/ebc833ea-1eeb-40a6-8f2f-7dcf9e85e3a6" /><br>

4️⃣ Кросс соединение (CROSS JOIN)
```sql
SELECT *
FROM movies.movie m
CROSS JOIN movies.genres mg
CROSS JOIN movies.genre g;
```
Выборка формирует декартово произведение — каждая строка из таблицы `movie (6 строк)` комбинируется с каждой строкой из таблицы `genres (15 строк)` и с каждой строкой из таблицы `genre (13 строк)` : 6 * 15 * 13 =  1170 строк в выборке
<img width="1193" height="1224" alt="image" src="https://github.com/user-attachments/assets/018afee8-dfdf-408c-afb9-fff13b3851fc" /><br>

5️⃣ Полное соединение (FULL OUTER JOIN)
```sql
SELECT *
FROM movies.movie m
FULL OUTER JOIN movies.genres mg ON m.id_movie = mg.id_movie
FULL OUTER JOIN movies.genre g on g.id_genre = mg.id_genre;
```
Выборка возвращает все записи из таблиц фильмов и жанров, независимо от наличия соответствия записей между ними
<img width="1174" height="740" alt="image" src="https://github.com/user-attachments/assets/1a1c4e96-9161-4d3d-9c7c-113f0bef17de" /><br>

6️⃣ Комбинированный запрос с несколькими типами соединений
```sql
SELECT *
FROM movies.movie m
CROSS JOIN movies.genres mg
LEFT JOIN movies.genre g on g.id_genre = mg.id_genre;
```
Выборка показывает список всех жанров из таблицы `genres` с каждым фильмом из таблицы `movie` и для каждой выбранной строки показывает название этого жанра из таблицы `genre`
<img width="1210" height="1218" alt="image" src="https://github.com/user-attachments/assets/49ccbf2e-e13b-42d2-a71d-b3618cf2e32c" /><br>


<b>Задание со ⭐</b><br>
Для реальной задачи "Мини-фильмотека" мне потребовались бы следующие запросы

1️⃣ Список всех фильмов со всеми существующими или несуществующими атрибутами, объединёнными в строки
```sql
SELECT table_genres.id_movie, name_movie, table_genres.year, list_genres, string_agg(name_country, ', ' ORDER BY mc.id_country) list_countries FROM (
   SELECT m.id_movie, name_movie, year, string_agg(name_genre, ', ' order by mg.id_genre) list_genres
   FROM movies.movie m
   LEFT JOIN movies.genres mg ON m.id_movie = mg.id_movie
   LEFT JOIN movies.genre g ON g.id_genre = mg.id_genre
   GROUP BY m.id_movie, name_movie) table_genres
LEFT JOIN movies.countries mc ON table_genres.id_movie = mc.id_movie
LEFT JOIN movies.country c ON c.id_country = mc.id_country
GROUP BY table_genres.id_movie, name_movie, year, list_genres;
```
<img width="1446" height="500" alt="image" src="https://github.com/user-attachments/assets/d6f7f0a9-d8ea-443f-8036-c83c1e4b604b" /><br>

2️⃣ Топ фильмов по средней пользовательской оценке
```sql
SELECT name_movie, year, round(sum(value)/count(value)::numeric, 2) aver_value
FROM movies.movie m
RIGHT JOIN movies."values" mv ON m.id_movie = mv.id_movie
GROUP BY name_movie, year
ORDER BY aver_value DESC;
```
<img width="802" height="419" alt="image" src="https://github.com/user-attachments/assets/89d7f131-48cc-4a21-86e5-a580cf027ee2" /><br>

3️⃣ Топ активных пользователей с максимальным количеством оценок и средней оценкой за фильмы
```sql
SELECT name_user, count(value) count_value, round(sum(value)/count(value)::numeric, 2) aver_value
FROM movies.users u
RIGHT JOIN movies."values" mv ON u.id_user = mv.id_user
GROUP BY name_user
ORDER BY count_value DESC, aver_value DESC;
```
<img width="980" height="544" alt="image" src="https://github.com/user-attachments/assets/79f52cf8-6086-46e5-ae24-6de87fdb22b2" /><br>










