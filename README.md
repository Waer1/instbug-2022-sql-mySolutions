## Problem 1:
Given 3 tables, students(id, name) , courses(id, name) , grades(id, course_id, student_id, grade),

find the top 100 students based on their

average grades sorted descendingly by the average grade and in case multiple students have the same average grade,
sort them lexicographically in ascending order by their names.

Your query should output a table with the following columns (name, average_grade).

#### Solution:
```SQL
SELECT students.name AS name,  AVG(grade)  AS average_grade
FROM grades
JOIN students ON students.id = student_id
GROUP BY student_id , students.name
ORDER BY AVG(grade) DESC, students.name
LIMIT 100;
```

___
## Problem 2:

For the same tables in **Problem 1**, for each student, get all the courses that he/she is enrolled in along with the grade he/she scored for each course.
Order the result by the student name in ascending order and if there is a tie, break it with the course name in ascending order and
if there is a tie break it with the grade in ascending order. The final result should have 3 columns with names (name, course, grade).

#### Solution:

```SQL
SELECT students.name AS name, courses.name AS course, grade AS grade
FROM grades, courses, students
where courses.id = course_id and students.id = student_id
GROUP BY student_id , students.name and course_id, courses.name,grade
ORDER BY students.name, courses.name, grade;
```

___

## Problem 3:
For the same tables in **Problem 1**, get the name of the most popular course (the one where the most students are enrolled) and if there is a tie, get the course that's lexicographically the smallest.
#### Solution:
```SQL
SELECT courses.name AS name
FROM grades
JOIN courses ON courses.id = course_id
GROUP BY course_id , courses.name
ORDER BY count(course_id) DESC, courses.name
LIMIT 1;

```
___
## Problem 4:
Given a table called "bugs" with the following columns (id, token, title, category, device, reported_at, created_at, updated_at).
Select all distinct bug categories.
#### Solution:
```SQL
SELECT DISTINCT category
FROM bugs;
```
___

## Problem 5:
For the same table in **Problem 4**, find how many bugs were created on "2019-03-01" or later. Your query should produce a table with one column called "count".
This problem is graded partially, 10% on correctness (your query gets the correct count) and 90% on performance (your query makes use of available indexes).

***Scored 10% only***
"Not Optimal solution"

```SQL
SHOW INDEXES FROM bugs;
```

```JSON
SHOW INDEXES FROM bugs;

"records": [
		{
			"Table": "bugs",
			"Non_unique": 0,
			"Key_name": "PRIMARY",
			"Seq_in_index": 1,
			"Column_name": "id",
			"Collation": "A",
			"Cardinality": 9791826,
			"Sub_part": null,
			"Packed": null,
			"Null": "",
			"Index_type": "BTREE",
			"Comment": "",
			"Index_comment": ""
		},
		{
			"Table": "bugs",
			"Non_unique": 1,
			"Key_name": "index_bugs_on_category_and_token_and_reported_at",
			"Seq_in_index": 1,
			"Column_name": "category",
			"Collation": "A",
			"Cardinality": 1,
			"Sub_part": null,
			"Packed": null,
			"Null": "YES",
			"Index_type": "BTREE",
			"Comment": "",
			"Index_comment": ""
		},
		{
			"Table": "bugs",
			"Non_unique": 1,
			"Key_name": "index_bugs_on_category_and_token_and_reported_at",
			"Seq_in_index": 2,
			"Column_name": "token",
			"Collation": "A",
			"Cardinality": 29946,
			"Sub_part": null,
			"Packed": null,
			"Null": "YES",
			"Index_type": "BTREE",
			"Comment": "",
			"Index_comment": ""
		},
		{
			"Table": "bugs",
			"Non_unique": 1,
			"Key_name": "index_bugs_on_category_and_token_and_reported_at",
			"Seq_in_index": 3,
			"Column_name": "reported_at",
			"Collation": "A",
			"Cardinality": 6085027,
			"Sub_part": null,
			"Packed": null,
			"Null": "YES",
			"Index_type": "BTREE",
			"Comment": "",
			"Index_comment": ""
		}
	]
}
```
```SQL
use instabug;
-- Step 1: Convert the target date to its corresponding id value
SET @targetDate = '2022-06-15';
SET @targetId := NULL;
SET @currentId := NULL;

-- Step 2-3: Perform binary search
SET @minId := NULL;
SET @maxId := NULL;
SET @currentId := NULL;

SELECT @minId:= MIN(id) , @maxId := MAX(id) FROM bugs;
SET @TotalBug = @maxId;

DROP PROCEDURE IF EXISTS While_Loop;

Delimiter //
CREATE PROCEDURE While_Loop()
   BEGIN
   WHILE @minId <= @maxId
   DO
	SET @currentId := @minId + ( @maxId - @minId ) DIV 2;
	-- SELECT 'current id is' , @currentId;

    IF EXISTS (
    SELECT 1
    FROM bugs
    WHERE id = @currentId
      AND created_at >= @targetDate
  ) THEN
    SET @maxId := @currentId - 1;
  ELSE
    SET @minId := @currentId + 1;
  END IF;
   END WHILE;
END //

CALL While_Loop();

select @TotalBug -  @currentId;
```

___

## Problem 6:

For the same table in **Problem 4**, find the title of the bug with token = "token660" and reported_at on "2020-08-30".
This problem is graded partially, 10% on correctness (your query gets the correct count) and 90% on performance (your query makes use of available indexes).

***Scored 10% only*** "Not Optimal Solution"
```SQL
SELECT DISTINCT category INTO categories FROM bugs;

SELECT title FROM bugs USE INDEX (index_bugs_on_category_and_token_and_reported_at)
WHERE category IN (SELECT * FROM categories) AND token = 'token660' AND reported_at = '2020-08-30';
```
