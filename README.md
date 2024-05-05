# Instagram--data--SQL--project

## Overview

This project is a multitude of SQL scripts with the scope of creating tables, inserting and rendering data. Below are mentioned only data retrieval scripts, as these are most important, the rest of them can be found in the SQL file.

## Data set
The dataset is from a fictive social media website, provided in the SQL course that I did at the beginning of my SQL journey, and contains with 7 tables: <br>
* users
* photos
* comments
* likes
* follows
* photo_tags
* tags


### Data model

![image](https://gist.github.com/assets/128899809/1884c3c6-177f-4a59-ba56-7ab09f7f3747)


## Data analysis questions:

1. Select top 5 customers

```
SELECT * FROM users 
ORDER BY created_at 
LIMIT 5;
```

Result:

| id  | username | created_at
| ---| --- | --- |
|80	 | Darby_Herzog    |	5/6/2016 10:14
|67	 | Emilio_Bernier52|	5/6/2016 23:04
|63	 | Elenor88	       |5/8/2016 11:30
|95	 | Nicole71	       |5/10/2016 3:30
|38	 | Jordyn.Jacobson2|5/14/2016 17:56

<br>

2. What is the month and year with the second highest number of people who registered on the platform?

````
SELECT
    DATE_FORMAT(created_at, '%b') AS Month,
	DATE_FORMAT(created_at, '%Y') AS Year,
	COUNT(id) AS user_nr
FROM users
GROUP BY
	Month,
	Year
ORDER BY 
user_nr DESC
LIMIT 1 OFFSET 1;
````
Result:

|Month |Year |user_nr
|--- |--- |---
|Aug | 2016	| 11

3. Which users never posted a photo? Select the users and provide a count of the total users 

````
SELECT
	IF(GROUPING(username), "All users", username) AS users,
    COUNT(username) AS count_user
FROM users 
LEFT JOIN photos 
ON 
users.id = photos.user_id 
WHERE photos.id IS NULL
GROUP BY 
	username
WITH ROLLUP;
````
|users |count_user
|--- |---
|Aniya_Hackett	|1
|Bartholome.Bernhard	|1
|Bethany20	|1
|Darby_Herzog	|1
|David.Osinski47	|1
|Duane60	|1
|Esmeralda.Mraz57	|1
|Esther.Zulauf61	|1
|Franco_Keebler64	|1
|Hulda.Macejkovic	|1
|Jaclyn81	|1
|Janelle.Nikolaus81	|1
|Jessyca_West	|1
|Julien_Schmidt	|1
|Kasandra_Homenick	|1
|Leslie67	|1
|Linnea59	|1
|Maxwell.Halvorson	|1
|Mckenna17	|1
|Mike.Auer39	|1
|Morgan.Kassulke|1
|Nia_Haag	    |1
|Ollie_Ledner37	|1
|Pearl7	        |1
|Rocio33	    |1
|Tierra.Trantow	|1
|All users	    |26

<br>

4.  What user and image has the most likes on the platform?

````
SELECT 
	username,
    photo_id,
    image_url,
	COUNT(photo_id) AS nr_likes
FROM photos
INNER JOIN likes ON photos.id = likes.photo_id
INNER JOIN users ON photos.user_id = users.id
GROUP BY photo_id
ORDER BY nr_likes DESC
LIMIT 1;
````

| username      | photo id | image_url           | nr_likes |
|---------------|----------|---------------------|----------|
| Zack_Kemmer93 | 145      | https://jarret.name | 48       |


5. How many times does the average user had posted between 2021 and 2022?

```
WITH count_posts
AS
(SELECT 
	user_id,
	COUNT(image_url) count_img
FROM photos
WHERE YEAR(created_at) 
BETWEEN "2021" AND "2022"
GROUP BY 
	user_id
)
SELECT 
	AVG(count_img)
FROM count_posts
; 
```

Result:

1.0909

<br>

6. How many photos posted each user?


```
SELECT 
	IF(GROUPING(username), "All users", username) AS users,
	COUNT(*) AS nr_photos
FROM photos 
INNER JOIN users 
	ON photos.user_id = users.id
GROUP BY 
username WITH ROLLUP
ORDER BY 
nr_photos DESC;
```

7. What are the top 5 most commonly used hashtags?

```
SELECT 
	tag_name, 
    COUNT(tag_id) AS tag_nr FROM tags 
INNER JOIN photo_tags 
	ON tags.id = photo_tags.tag_id
GROUP BY tag_name
ORDER BY tag_nr DESC
LIMIT 5;
```

| tag_name | tag_nr |
|----------|--------|
| smile    | 59     |
| party    | 39     |
| fun      | 38     |
| concert  | 24     |
| beach    | 42     |

<br>

8. Check if there are users who liked every photo from the website.

```
SELECT 
	users.username, 
    count(*) likes_count
FROM users
JOIN likes 
	ON users.id = likes.user_id
GROUP BY username
HAVING likes_count = (
					SELECT 
						COUNT(DISTINCT(id))
					FROM photos
);
```

9. Create a trigger to ensure people cannot follow themselves.

```
-- 1. Create the trigger 

DELIMITER //

CREATE TRIGGER must_not_follow_themselves
	BEFORE INSERT ON follows 
	FOR EACH ROW
	BEGIN
					IF NEW.followee_id = NEW.follower_id THEN
							SIGNAL SQLSTATE "45000"
								SET MESSAGE_TEXT = "Must not follow themselves";
					ELSEIF followee_id = NEW.follower_id THEN
							SIGNAL SQLSTATE "45000"
								SET MESSAGE_TEXT = "Must not follow themselves";
					ELSEIF NEW.followee_id = follower_id THEN
							SIGNAL SQLSTATE "45000"
								SET MESSAGE_TEXT = "Must not follow themselves";
                    
					END IF;
	END;//

DELIMITER ;

-- 2. Test it 

INSERT INTO follows(follower_id, followee_id, created_at)
VALUES (1, 1, NOW());

-- 3. Drop trigger

DROP TRIGGER ig_clone.must_not_follow_themselves;
```

