# SQL_Analysis_on_Famous_Paintings
SQL_Analysis_on_Famous_Paintings
SELECT * FROM artist;
SELECT * FROM canvas_size;
SELECT * FROM image_link;
SELECT * FROM museum;
SELECT * FROM museum_hours;
SELECT * FROM product_size;
SELECT * FROM subject;
SELECT * FROM work;

1.	Are there museums without any paintings?

SELECT m.museum_id, count(w.name) AS nop 
FROM museum m 
LEFT JOIN work w on m.museum_id = w.museum_id
GROUP BY m.museum_id
HAVING nop ISNULL;

2.	Fetch all the paintings which are not displayed on any Museum?
SELECT *
FROM work 
WHERE museum_id ISNULL;
3.	 How many paintings have an asking price of more than their regular price?
SELECT COUNT(*) AS Markedup_Total 
FROM product_size 
WHERE sale_price > regular_price;

4.	Identify the paintings whose asking price is less than 50% of its regular price?
SELECT *
FROM product_size 
WHERE sale_price < (regular_price/2);

5.	Identify the museums with invalid city information in the given dataset
SELECT *
FROM museum
WHERE city REGEXP '^[0-9]';

6.	 Fetch the top 10 most famous painting subject
SELECT subject, count(work_id) AS nos
FROM subject
GROUP BY subject
ORDER BY nos DESC
LIMIT 10;
7.	 Identify the museums which are open on both Sunday and Monday. Display museum name, city.
WITH cte AS
(SELECT museum_id, COUNT(day) AS cnt 
FROM (
SELECT * FROM museum_hours 
WHERE day IN ('Sunday','Monday'))
GROUP BY museum_id 
HAVING cnt=2)
SELECT m.name,m.city FROM museum m 
WHERE m.museum_id IN 
(SELECT cte.museum_id FROM cte);

8.	 How many museums are open every single day?
SELECT COUNT(a.museum_id) FROM 
(SELECT museum_id, COUNT(museum_id) FROM museum_hours
GROUP BY museum_id HAVING COUNT(day)=7) a;
(Or)
SELECT  COUNT(*) AS everyday_museums
FROM(
SELECT *,  ROW_NUMBER() OVER(PARTITION BY museum_id) AS rn
  	FROM museum_hours) AS x
  WHERE x.rn = 7;

9.	Which are the top 5 most popular museum? (Popularity is defined based on most no of paintings in a museum)
SELECT m.museum_id, m.name, m.country,
COUNT(w.work_id) AS No_Of_Paintings 
FROM museum m
JOIN work w on m.museum_id = w.museum_id
GROUP BY m.museum_id 
ORDER BY No_Of_Paintings DESC
LIMIT 5;

10.	 Who are the top 5 most popular artist? (Popularity is defined based on most no of paintings done by an artist)
SELECT a.artist_id, a.full_name, a.nationality, 
COUNT(w.work_id) AS No_Of_Paintings
FROM artist a 
JOIN work w on a.artist_id = w.artist_id
GROUP BY w.artist_id
ORDER BY No_Of_Paintings DESC
LIMIT 5;

11.	 Which museum is open for the longest during a day. Display museum name, state and hours open and which day?
SELECT * FROM (
	SELECT m.name, m.state, mh.day,
		TO_TIMESTAMP(open, ‘HH:MI AM’) AS open_time,
		TO_TIMESTAMP(close, ‘HH:MI PM’) AS close_time,
		(TO_TIMESTAMP(close, ‘HH:MI PM’) – 
TO_TIMESTAMP(open, ‘HH:MI AM’)) AS duration, 
RANK OVER( ORDER BY (TO_TIMESTAMP(close, ‘HH:MI PM’) – 
TO_TIMESTAMP(open, ‘HH:MI AM’)) DESC) as rnk
FROM museum_hours mh
JOIN museum m on m.museum_id=mh.museum_id) a
WHERE a.rnk=1;

12.	Which museum has the most painting style?
SELECT w.museum_id, m.name, m.country, 
COUNT(DISTINCT w.style) AS styles_count 
FROM work w 
JOIN museum m ON m.museum_id = w.museum_id
GROUP BY w.museum_id
ORDER BY styles_count DESC
LIMIT 1;


13.	Which museum has the most no. of most popular painting style?
SELECT m.museum_id,m.name, w.style,count(w.work_id) AS no_of_paintings
FROM work w 
JOIN museum m ON m.museum_id=w.museum_id
WHERE style=(
	SELECT style  
	FROM work GROUP BY style
	ORDER BY COUNT(work_id) DESC LIMIT 1)
GROUP BY m.museum_id
ORDER BY no_of_paintings DESC
LIMIT 1;

14.	 Identify the top 5 artists whose paintings are displayed in multiple countries.
SELECT w.artist_id, a.full_name, a.nationality, 
	COUNT(DISTINCT m.country) AS No_of_countries, 
	COUNT(w.work_id) AS No_of_paintings 
FROM work w 
JOIN museum m on m.museum_id= w.museum_id
JOIN artist a on w.artist_id = a.artist_id
GROUP BY w.artist_id
ORDER BY No_of_paintings DESC
LIMIT 5;


15.	 Which country has the 5th highest no of paintings?
SELECT * FROM(
SELECT m.country, count(w.work_id) AS no_of_paintings, 
RANK() OVER(ORDER BY COUNT(w.work_id) DESC) AS rnk 
FROM work w
JOIN museum m ON m.museum_id= w.museum_id
GROUP BY m.country
ORDER BY no_of_paintings DESC) a 
WHERE a.rnk=5;
(or)
SELECT m.country, count(*) AS no_of_paintings
FROM artist a
JOIN work w ON a.artist_id= w.artist_id
JOIN museum m ON m.museum_id = w.museum_id
GROUP BY m.country
ORDER BY COUNT(*) DESC
LIMIT 1
OFFSET 4;

16.	Which are the 3 most popular and 3 least popular painting styles?
SELECT * FROM(
	SELECT style, COUNT(work_id) AS no_of_paintings,
 'Most Popular' AS status 
	FROM work 
	GROUP BY style
	ORDER BY COUNT(work_id) DESC
	LIMIT 3)
UNION 
SELECT * FROM (
	SELECT style, count(work_id) AS no_of_paintings,
 'Least Popular' AS status
	FROM work 
	GROUP BY style
	ORDER BY COUNT(work_id)
	LIMIT 3)
ORDER BY no_of_paintings DESC;

17.	 Which artist has the most no. of Portraits paintings outside USA?. Display artist name, no of paintings and the artist nationality.

SELECT w.artist_id, a.full_name, a.nationality, 
COUNT(w.work_id) AS no_of_portraits 
FROM work w 
JOIN subject s on w.work_id=s.work_id
JOIN museum m on m.museum_id=w.museum_id
JOIN artist a on a.artist_id=w.artist_id
WHERE s.subject='Portraits' AND m.country != 'USA'
GROUP BY w.artist_id, a.full_name, a.nationality
ORDER BY no_of_portraits DESC
LIMIT 2;

