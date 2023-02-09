---
layout: post
title: "Оконные функции"
date: 2019-04-19 12:00:00 +0300
categories: sql
permalink: window-functions
---

# Оконные функции

Оконные функции используются для задач, в которых необходимо получать значения агрегированных функций, например, MAX, SUM, AVG и тд.

> Большинство современных CУБД поддерживают оконные функции, кроме MySQL

## Пример задачи

Выберите все названия книг, автора, максимальную и минимальную цену книги по каждому автору

## Создаем таблицу

````sql
DROP TABLE IF EXISTS books;

CREATE TABLE books (
  id serial PRIMARY KEY,
  name varchar NOT NULL,
  price integer NOT NULL,
  author_id bigint NOT NULL
);

INSERT INTO books ("name", "price", "author_id") VALUES
('First', 10, 1),
('Second', 20, 2),
('Third', 30, 1),
('Forth', 40, 2),
('Fifth', 50, 1);
````

## Запрос c GROUP BY

````sql
SELECT author_id, MAX(price) as max_price, MIN(price) as min_price
FROM books
GROUP BY author_id
````

Проблема запроса с GROUP BY в том, что запрос производит одну строчку для каждой группы! То есть должно быть выбрано то, что однозначно вычисляется для группы строк. Невозможно выбрать название отдельной книги!

**Минусы GROUP BY**

- GROUP BY "уничтожает" отдельные строки
- Можно узнать только признак группировки, и значение агрегатных функций

При использовании GROUP BY, для решения данной задачи придется использовать дополнительный подзапрос, например, так:

````sql
WITH tmp AS (
	SELECT author_id, MAX(price) as max_price, MIN(price) as min_price
	FROM books
	GROUP BY author_id
)
SELECT b.name, b.author_id, tmp.max_price, tmp.min_price FROM books AS b
JOIN tmp ON b.author_id = tmp.author_id
````

## Запрос с OVER

````sql
SELECT name, author_id,
MAX(price) OVER(PARTITION BY author_id) AS max_price,
MIN(price) OVER(PARTITION BY author_id) AS min_price
FROM books;
````