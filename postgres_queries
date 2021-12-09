/* 1). Выберите список всех комментариев, созданных пользователем с идентификатором 1. 
Поля для вывода: id, created_at, offer_id, comment_text.*/

SELECT id, created_at, user_id, offer_id, comment_text 
FROM comments
WHERE user_id = 1;

/* 2) Выведите список объявлений (id, created_at, user_id, offer_type, title, 
price, picture), опубликованных в октябре 2021 года с сортировкой по дате публикации 
от самых свежих к более поздним. Дату публикации выведите в формате ‘DD.MM.YYYY’.*/

SELECT id, TO_CHAR(created_at::DATE, 'DD.MM.YYYY'), user_id, offer_type, title, price, picture
FROM offers
WHERE created_at >= '2021-10-01'
		AND created_at < '2021-11-01'
ORDER BY created_at DESC;

/* 3) Выберите список пользователей, которые ещё не опубликовали ни одного объявления. Поля для вывода: идентификатор пользователя, 
email, дата регистрации, имя и фамилия одной строкой как ‘user_name’. Отсортируйте по возрастанию даты регистрации. */

SELECT u.id, email, u.created_at, first_name || ' ' || last_name AS user_name
FROM users u
LEFT JOIN offers o ON u.id = o.user_id
WHERE user_id ISNULL
ORDER BY created_at;

/* 4) Выберите среди всех объявлений на продажу самые дорогие товары, их количество динамическое и заранее неизвестно.
Выведите их идентификаторы, автора (имя, фамилия), заголовки и цену продажи. */

CREATE OR REPLACE FUNCTION get_most_expensive(count int) 
RETURNS TABLE (id int, author text, title varchar, price numeric) 
AS $$
BEGIN
	RETURN QUERY
		SELECT o.id, u.first_name||' '||u.last_name as author, o.title, o.price
		FROM offers o
		LEFT JOIN users u ON u.id = o.user_id
		ORDER BY o.price DESC
		LIMIT count;
END;
$$ LANGUAGE plpgsql;

SELECT * FROM get_most_expensive(5);

/* 5) Для вывода на сайте выберите список всех категорий, в которых есть хотя бы одно объявление с указанием количества объявлений по каждой категории. 
Выведите id категории, title, slug, количество объявлений (offer_amount).*/

SELECT category_id, c.title, c.slug, COUNT(*) AS offer_amount
FROM category_offer co
LEFT JOIN categories c ON  c.id = co.category_id
LEFT JOIN offers o ON o.id = co.offer_id 
GROUP BY category_id, c.title, c.slug;

/*1) Выберите список объявлений, относящихся к категории “Дом”. 
Поля: название категории (category_name) и данные объявления (id, created_at, user_id, title, price). Список категорий для объявлений выводить не нужно.
Выведите следующие 4 объявления после первых 4.*/
SELECT o.id, o.created_at, o.user_id, o.title, o.price, c.title AS category_name
FROM offers o
LEFT JOIN category_offer co ON co.offer_id = o.id
LEFT JOIN categories c ON c.id = co.category_id
WHERE c.title = 'Дом'
LIMIT 4 OFFSET 4;

/* 2) Выведите объявления (id, title, user_id, offer_type, price) со всеми категориями, к которым оно относится, 
собранными в одну строку, например: “Спорт, Развлечения, Дети”. Назовите этот столбец categories. Отсортируйте по
убыванию цены. Выведите названия типов объявлений на русском: если тип buy, то нужно вывести “Куплю”, если sell - 
“Продам”, соответственно. */

SELECT offer_id, o.title, o.user_id, 
CASE o.offer_type 
	WHEN 'buy' THEN 'Куплю'
	WHEN 'sell' THEN 'Продам' END AS offer_type, 
	o.price, array_to_string(ARRAY_AGG(c.title), ' ,') AS categories
FROM category_offer co
LEFT JOIN categories c ON c.id = co.category_id
LEFT JOIN offers o ON o.id = co.offer_id
GROUP BY offer_id, o.title, o.user_id, o.offer_type, o.price
ORDER BY o.price DESC;

/*3) Выведите список пользователей (id, first_name, last_name, email), 
количество объявлений, созданных ими (offer_amount), и количество 
комментариев под этими объявлениями (comments_amount). 
Результат отсортируйте по убыванию  offer_amount.*/

SELECT u.id AS user_id, u.first_name, u.last_name, u.email, 
	   COUNT(sq.order_id) AS offer_amount, 
	   COALESCE(SUM(comments_count), 0) AS comments_amount 
FROM users u
LEFT JOIN 
	(SELECT o.id AS order_id, o.user_id,  COUNT(c.id) AS comments_count 
	 FROM offers o
	 LEFT JOIN comments c ON o.id = c.offer_id
	 GROUP BY o.id) AS sq ON u.id = sq.user_id
GROUP BY u.id
ORDER BY offer_amount DESC;

/*4) Реализуйте запрос поиска: 
Выберите все объявления с типом “Куплю” (“buy”) в категории “Животные”, 
в заголовке которых есть слова “кролик” и “гараж” одновременно. 
Полный текст объявления обрежьте до 30 символов, добавьте к полученной 
строке “...” и назовите announce. Отберите объявления с ценой менее 50000.
Поля для вывода: идентификатор объявления, тип, категория, автор (имя, фамилия), заголовок, анонс, стоимость*/

SELECT o.id AS offer_id, 
	   o.offer_type, 
	   c.title AS category_title, 
	   u.first_name ||' '|| u.last_name AS author,
	   o.title AS offer_title, 
	   SUBSTRING(o.full_text FOR 30)||'...' AS announce, 
	   o.price
FROM offers o 
LEFT JOIN category_offer co ON co.offer_id = o.id
LEFT JOIN categories c ON c.id = co.category_id
LEFT JOIN users u ON u.id = o.user_id
WHERE  offer_type = 'buy' 
		AND c.title = 'Животные' 
		AND o.title ILIKE ALL(ARRAY['%гараж%', '%кролик%'])
		AND o.price < 50000;

/*5) Напишите запрос, который по массиву идентификаторов категорий соберёт массив названий соответствующих категорий. 
Пример: на входе массив ARRAY[1, 2, 5, 7], на выходе  массив строк: */

CREATE OR REPLACE FUNCTION get_categories_array(category_ids INT[]) 
RETURNS VARCHAR[] AS $$
BEGIN
	RETURN ARRAY 
		(SELECT title FROM categories WHERE id = ANY(category_ids));
END;
$$ LANGUAGE plpgsql;

SELECT * FROM get_categories_array(ARRAY[1,2,5,7]);

/*6) Соберите мини-отчёт: реализуйте выборку количества объявлений по месяцам 2021 года. 
Поля для вывода: year, monthname, offers_amount. Отсортируйте отчёт по месяцам по возрастанию.  */

SELECT  to_char(created_at, 'YYYY') AS year, 
	to_char(created_at, 'Month') AS monthname, 
	COUNT(*) AS offers_count
FROM offers
GROUP BY monthname, year, date_part('month', created_at)
HAVING to_char(created_at, 'YYYY') = '2021'
ORDER BY year, date_part('month', created_at);

/* 7) Реализуйте запрос из пункта 6 с добавлением нарастающего итога по месяцам. Те же столбцы + столбец offers_sum. */

SELECT to_char(created_at, 'YYYY') AS year, 
	to_char(created_at, 'Month') AS monthname, 
	COUNT(*) as offers_count,
	SUM(COUNT(*)) OVER(ORDER BY to_char(created_at, 'YYYY'), date_part('month', created_at)) AS offers_sum
FROM offers
GROUP BY monthname, year, date_part('month', created_at)
HAVING to_char(created_at, 'YYYY') = '2021'
ORDER BY year, date_part('month', created_at);

/*8) Соберите jsonb-массив всех комментариев для объявления с id 7. 
Каждый комментарий должен быть представлен jsonb-объектом со следующими данными: 
id комментария, текст, дата создания, id пользователя, создавшего комментарий,
фамилия и имя одной строкой, ссылка на аватар.*/

SELECT to_jsonb(ARRAY_AGG(com)) FROM 
	(SELECT c.id, c.comment_text, c.created_at, c.user_id, u.first_name||' '||u.last_name AS username, u.avatar
	 FROM comments c
	 JOIN users u ON u.id = c.user_id
	 WHERE u.id = 7) AS com

/*9) Создайте pl/pgsql-функцию, выполняющую запрос из п. 4.
Функция должна принимать набор параметров-фильтров для поиска.
Функция должна возвращать набор строк - список объявлений, соответствующих фильтрам. */

/* Функция для добавления маски %% к фильтрам */
CREATE OR REPLACE FUNCTION add_mask(filters varchar[]) 
RETURNS varchar[] AS $$
BEGIN
  FOR i IN 1..array_length(filters, 1)
  LOOP
    filters[i] := '%'||filters[i]||'%';
  END LOOP;
  RETURN filters;
END;
$$ LANGUAGE plpgsql;

/* Функция поиска объявлений по фильтрам */
CREATE OR REPLACE FUNCTION filter_offers(filters varchar[]) 
RETURNS TABLE (offer_id int, offer_type offer_type,  category_title varchar,  author text,  offer_title varchar, announce text, price numeric) 
AS $$
	DECLARE
	arr varchar[] = add_mask(filters);
BEGIN
	RETURN QUERY
		SELECT o.id AS offer_id, 
	    o.offer_type as offer_type, 
	    c.title AS category_title, 
	    u.first_name ||' '|| u.last_name AS author,
	    o.title AS offer_title, 
	    SUBSTRING(o.full_text FOR 30)||'...' AS announce, 
	    o.price AS price
FROM offers o 
LEFT JOIN category_offer co ON co.offer_id = o.id
LEFT JOIN categories c ON c.id = co.category_id
LEFT JOIN users u ON u.id = o.user_id
WHERE  o.offer_type = 'buy' 
		AND c.title = 'Животные' 
		AND o.title ILIKE ALL(arr)
		AND o.price < 50000;
END;
$$ LANGUAGE plpgsql;
