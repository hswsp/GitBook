# 02 - Modern SQL

![0001.jpg](assets/1677724420254-796ab31f-c5cd-4618-af7c-bc242a32dbbc.jpeg)

![0002.jpg](assets/1677724419262-5d7a224f-0b12-4726-bf5e-60c15b9a8bc4.jpeg)

# Relational Languages

Edgar Codd published a major paper on relational models in the early 1970s. Originally, he only deﬁned the mathematical notation for how a DBMS could execute queries on a relational model DBMS.
The user only needs to specify the result that they want using a declarative language (i.e., SQL). The DBMS is responsible for determining the most efﬁcient plan to produce that answer.
Relational algebra is based on **sets** (unordered, no duplicates). SQL is based on **bags** (unordered, allows duplicates).

# SQL History

Declarative query language for relational databases. It was originally developed in the 1970s as part of the IBM **System R** project. IBM originally called it “SEQUEL” (Structured English Query Language). The name changed in the 1980s to just “SQL” (Structured Query Language).

The language is comprised of different classes of commands:

1. **Data Manipulation Language (DML):**  SELECT, INSERT, UPDATE, and DELETE statements.
2. **Data Deﬁnition Language (DDL): **Schema deﬁnitions for tables, indexes, views, and other objects.
3. **Data Control Language (DCL): **Security, access controls.

![0003.jpg](assets/1677724419724-2a3215c5-e610-446a-8a3e-64b8cd606161.jpeg)

![0004.jpg](assets/1677724420411-8f8218c4-2125-4d94-9c69-baf59d0acfc7.jpeg)

SQL is not a dead language. It is being updated with new features every couple of years. SQL-92 is the minimum that a DBMS has to support to claim they support SQL. Each vendor follows the standard to a certain degree but there are many proprietary extensions.

![0005.jpg](assets/1677724420175-e3ea27e8-237a-474e-860c-56051b14e753.jpeg)

Some of the major updates released with each new edition of the SQL standard are shown below.

- SQL:1999 Regular expressions, Triggers
- SQL:2003 XML, Windows, Sequences
- SQL:2008 Truncation, Fancy sorting
- SQL:2011 Temporal DBs, Pipelined DML
- SQL:2016 JSON, Polymorphic tables

![0006.jpg](assets/1677724424080-39024196-4b10-47f2-91c3-959abee768b2.jpeg)

![0007.jpg](assets/1677724422979-07eb45aa-18ea-4fcd-92ef-76ab26d070bd.jpeg)

![0008.jpg](assets/1677724422730-619c9772-ed77-49b0-91cf-3e825a1823a9.jpeg)

# Joins

Combines columns from one or more tables and produces a new table. Used to express queries that involve data that spans multiple tables.
Example: *Which students got an A in 15-721?*

```SQL
CREATE TABLE student (
  sid INT PRIMARY KEY,
  name VARCHAR(16),
  login VARCHAR(32) UNIQUE,
  age SMALLINT,
  gpa FLOAT
);

CREATE TABLE course (
  cid VARCHAR(32) PRIMARY KEY,
  name VARCHAR(32) NOT NULL
);

CREATE TABLE enrolled (
  sid INT REFERENCES student (sid),
  cid VARCHAR(32) REFERENCES course (cid),
  grade CHAR(1)
);
```

![0009.jpg](assets/1677724424020-51e12686-baf9-442f-9c26-b80069a76549.jpeg)

```SQL
SELECT s.name
	FROM enrolled AS e, student AS s
WHERE e.grade = 'A' AND e.cid = '15-721'
	AND e.sid = s.sid;
```

# Aggregates

An aggregation function takes in a bag of tuples as its input and then produces a single scalar value as its output. **Aggregate functions can (almost) only be used in a SELECT output list.**

- AVG(COL): The average of the values in COL
- MIN(COL): The minimum value in COL
- MAX(COL): The maximum value in COL
- COUNT(COL): The number of tuples in the relation

![0010.jpg](assets/1677724424065-0f0a4357-0ffc-4d1d-b5ba-7da36b4b417b.jpeg)

Example: *Get # of students with a ‘@cs’ login*.
The following three queries are equivalent:

```SQL
SELECT COUNT(*) FROM student WHERE login LIKE '%@cs';

SELECT COUNT(login) FROM student WHERE login LIKE '%@cs';

SELECT COUNT(1) FROM student WHERE login LIKE '%@cs';
```

![0011.jpg](assets/1677724426726-8c4e625a-bb7e-4744-9b53-a86bf4af7169.jpeg)

**A single SELECT statement can contain multiple aggregates**:
Example: *Get # of students and their average GPA with a ‘@cs’ login*.

```SQL
SELECT AVG(gpa), COUNT(sid) FROM student WHERE login LIKE '%@cs';
```

![0012.jpg](assets/1677724426476-f4f546aa-f54a-4b86-b43e-89853faf71aa.jpeg)

Some aggregate functions (e.g. COUNT, SUM, AVG) support the DISTINCT keyword:
Example: *Get # of unique students and their average GPA with a ‘@cs’ login.*

```SQL
SELECT COUNT(DISTINCT login) FROM student WHERE login LIKE '%@cs';
```

![0013.jpg](assets/1677724427478-2a05f988-6ace-4ddc-85c0-fa3c58fab6b3.jpeg)

Output of other columns outside of an aggregate is **undeﬁned** (e.cid is undeﬁned below).
Example: *Get the average GPA of students in each course.*

```SQL
SELECT AVG(s.gpa), e.cid FROM enrolled AS e, student AS s WHERE e.sid = s.sid;
```

![0014.jpg](assets/1677724427718-01d0dc32-1f9e-4961-b63f-3318df99d4f8.jpeg)

![0015.jpg](assets/1677724427981-896ae9da-bcd1-4663-bced-9f3d0feb384e.jpeg)

Non-aggregated values in SELECT output clause must appear in GROUP BY clause.

```SQL
SELECT AVG(s.gpa), e.cid 
	FROM enrolled AS e, student AS s
WHERE e.sid = s.sid
	GROUP BY e.cid;
```

![0016.jpg](assets/1677724430542-b1719d69-7071-4c70-bc8f-21d452506661.jpeg)

![0017.jpg](assets/1677724430156-7be416bf-cc9b-4733-a073-98990b29fa6d.jpeg)

![0018.jpg](assets/1677724431209-e1688f6f-de21-4017-a3fa-5aea994afc89.jpeg)

![0019.jpg](assets/1677724431171-73a12877-4620-41d0-a285-db7b77724c51.jpeg)

The HAVING clause ﬁlters output results based on aggregation computation. This make HAVING behave like a WHERE clause for a GROUP BY.
Example: *Get the set of courses in which the average student GPA is greater than 3.9.*

```SQL
SELECT AVG(s.gpa) AS avg_gpa, e.cid
	FROM enrolled AS e, student AS s
WHERE e.sid = s.sid
	GROUP BY e.cid
HAVING avg_gpa > 3.9;
```

**The above query syntax is supported by many major database systems, but is not compliant with the SQL standard**. To make the query standard compliant, we must repeat use of AVG(S.GPA) in the body of the HAVING clause.

```SQL
SELECT AVG(s.gpa), e.cid
	FROM enrolled AS e, student AS s
WHERE e.sid = s.sid
	GROUP BY e.cid
HAVING AVG(s.gpa) > 3.9;
```

![0020.jpg](assets/1677724431704-da20dab9-b865-4f39-94a3-b089299c7453.jpeg)

![0021.jpg](assets/1677724433635-5ed34174-9f4a-4b09-9312-62ff95a85a2b.jpeg)

![0022.jpg](assets/1677724433954-55a9f4c7-f86e-4009-b113-7b639c7751cf.jpeg)

![0023.jpg](assets/1677724435137-7d703594-d78a-48b1-80d4-5dd6548c1c60.jpeg)

# String Operations

The SQL standard says that strings are **case sensitive **and **single-quotes only**. There are functions to manipulate strings that can be used in any part of a query.

![0024.jpg](assets/1677724436180-c547d54c-68d9-43fa-9cff-990ab6ef01da.jpeg)

**Pattern Matching: **The LIKE keyword is used for string matching in predicates.

- “%” matches any substrings (including empty).
- “_” matches any one character.

![0025.jpg](assets/1677724436173-eb4ad929-28e7-40c2-8e95-a3060a38e7c8.jpeg)

**String Functions SQL-92** deﬁnes string functions. Many database systems implement other functions in addition to those in the standard. Examples of standard string functions include `SUBSTRING(S, B, E)` and `UPPER(S)`.

![0026.jpg](assets/1677724437528-2b643692-2c4f-4a78-b7fe-af0554f939b6.jpeg)

**Concatenation:**  Two vertical bars (“||”) will concatenate two or more strings together into a single string.

![0027.jpg](assets/1677724437736-23e8e069-e239-40bc-886b-71acd9bf9db1.jpeg)

# Date and Time

Operations to manipulate DATE and TIME attributes. Can be used in either output or predicates. The speciﬁc syntax for date and time operations varies wildly across systems.

![0028.jpg](assets/1677724438581-c13a6005-b201-4cb4-a073-aa702066c0fb.jpeg)

# Output Redirection

Instead of having the result a query returned to the client (e.g., terminal), you can tell the DBMS to store the results into another table. You can then access this data in subsequent queries.

- **New Table:**  Store the output of the query into a new (permanent) table.

```SQL
SELECT DISTINCT cid INTO CourseIds FROM enrolled;
```

- **Existing Table**: Store the output of the query into a table that already exists in the database. The target table must have the same number of columns with the same types as the target table, but the **names of the columns in the output query do not have to match**.

```SQL
INSERT INTO CourseIds (SELECT DISTINCT cid FROM enrolled);
```

![0029.jpg](assets/1677724440116-e53ab8bc-78b9-44b8-a0e7-b518e5c5d754.jpeg)

![0030.jpg](assets/1677724440132-55184289-c45d-4485-8528-a9189079abff.jpeg)

![0031.jpg](assets/1677724441698-833e154d-d1da-4ed5-85e0-2d50ce0c9ba7.jpeg)

# Output Control

Since results SQL are unordered, we must use the ORDER BY clause to impose a sort on tuples:

```SQL
SELECT sid, grade FROM enrolled WHERE cid = '15-721'
	ORDER BY grade;
```

The default sort order is ascending (ASC). We can manually specify DESC to reverse the order:

```SQL
SELECT sid, grade FROM enrolled WHERE cid = '15-721'
	ORDER BY grade DESC;
```

![0032.jpg](assets/1677724441710-461eb6b2-ffe5-47ba-a157-9e47bc61af8a.jpeg)

We can use multiple ORDER BY clauses to break ties or do more complex sorting:

```SQL
SELECT sid, grade FROM enrolled WHERE cid = '15-721'
ORDER BY grade DESC, sid ASC;
```

![0033.jpg](assets/1677724442687-a68da143-5dcb-474a-a80b-e3a0667f9752.jpeg)

We can also use any arbitrary expression in the ORDER BY clause:

```SQL
SELECT sid FROM enrolled WHERE cid = '15-721'
	ORDER BY UPPER(grade) DESC, sid + 1 ASC;
```

![0034.jpg](assets/1677724444221-99df1a66-93de-4933-8e2c-e9537fb2e16d.jpeg)

By default, the DBMS will return all of the tuples produced by the query. We can use the LIMIT clause to restrict the number of result tuples:

```SQL
SELECT sid, name FROM student WHERE login LIKE '%@cs'
	LIMIT 10;
```

We can also provide an **offset **to return a range in the results:

```SQL
SELECT sid, name FROM student WHERE login LIKE '%@cs'
	LIMIT 20 OFFSET 10;
```

Unless we use an ORDER BY clause with a LIMIT, **the DBMS may produce different tuples in the result **on each invocation of the query because the relational model does not impose an ordering.

![0035.jpg](assets/1677724443900-2d759ba8-5e91-41f2-a43c-e4f8f60b9fa3.jpeg)

![0036.jpg](assets/1677724445639-3983a837-d6f7-466c-9e1e-2ac9ac377b00.jpeg)

# Nested Queries

Invoke queries inside of other queries to execute more complex logic within a single query. Nested queries are often difﬁcult to optimize.
The scope of outer query is included in an inner query (i.e. the **inner query can access attributes from outer query**), but not the other way around.
Inner queries can appear in almost any part of a query:

1. SELECT Output Targets:

```SQL
SELECT (SELECT 1) AS one FROM student;
```

1. FROM Clause:

```SQL
SELECT name
  FROM student AS s, (SELECT sid FROM enrolled) AS e
  WHERE s.sid = e.sid;
```

1. WHERE Clause:

```SQL
SELECT name FROM student
	WHERE sid IN ( SELECT sid FROM enrolled );
```

![0037.jpg](assets/1677724445219-7d6f764c-72f9-4bf8-b0cc-980a7fd986bc.jpeg)

Example: *Get the names of students that are enrolled in ‘15-445’.*

```SQL
SELECT name FROM student
WHERE sid IN (
SELECT sid FROM enrolled
WHERE cid = '15-445'
);
```

Note that *sid* has different scope depending on where it appears in the query.

![0038.jpg](assets/1677724445743-b316840b-49e3-47e4-8e13-ab063d608665.jpeg)

![0039.jpg](assets/1677724446992-0ff3019a-6394-460e-8117-39c957be3f00.jpeg)

![0040.jpg](assets/1677724447507-6692ddae-43d2-4141-8563-0530caee2749.jpeg)

Example: *Find student record with the highest id that is enrolled in at least one course.*

```SQL
SELECT student.sid, name
  FROM student
  JOIN (SELECT MAX(sid) AS sid
  	FROM enrolled) AS max_e
  ON student.sid = max_e.sid;
```

## Nested Query Results Expressions:

- ALL: Must satisfy expression for all rows in sub-query.
- ANY: Must satisfy expression for at least one row in sub-query.
- IN: Equivalent to =`ANY()`.
- EXISTS: At least one row is returned.

Example: *Find all courses that have no students enrolled in it.*

```SQL
SELECT * FROM course
  WHERE NOT EXISTS(
  	SELECT * FROM enrolled
  		WHERE course.cid = enrolled.cid
);
```

![0041.jpg](assets/1677724448569-f3896532-d71e-4c2b-aa34-ff90adefe808.jpeg)

![0042.jpg](assets/1677724448530-0250a640-5a4e-4dfc-be37-d165f200855f.jpeg)

![0043.jpg](assets/1677724449823-ac5178fe-fb06-4718-a1fe-0efd821fdea1.jpeg)

![0044.jpg](assets/1677724450104-557b734a-6a45-474e-9296-dffba9191055.jpeg)

![0045.jpg](assets/1677724451556-1a5d80cd-de37-4278-a97b-13e46a368b15.jpeg)

![0046.jpg](assets/1677724451699-84c7630d-74fa-4d9a-a90e-0b5f61ecb6cc.jpeg)

![0047.jpg](assets/1677724452103-c96ee36e-5e16-4ae5-b6b0-61c574fd774c.jpeg)

![0048.jpg](assets/1677724453834-66186baa-f267-4599-afeb-60aaab6d4ea7.jpeg)

![0049.jpg](assets/1677724453864-b7a5c4b0-aab3-4538-b919-6e8efd3c885a.jpeg)

![0050.jpg](assets/1677724454362-7f051dc1-72b5-462c-a2df-ec57a3cac38c.jpeg)

![0051.jpg](assets/1677724455390-38793a85-e8e2-4646-851e-a38bbdf71946.jpeg)

# Window Functions

A window function perform “sliding” calculation across a set of tuples that are related. Like an aggregation but tuples are not grouped into a single output tuple.
**Functions: **The window function can be any of the aggregation functions that we discussed above. There are also also special window functions:

1. `ROW_NUMBER`: The number of the current row.
2. `RANK`: The order position of the current row.

![0052.jpg](assets/1677724455887-489441fd-ff8d-4be8-9ead-115329bca42e.jpeg)

![0053.jpg](assets/1677724457838-cb5d6975-dcb7-4ac5-8e96-78116636e837.jpeg)

**Grouping:**  The OVER clause speciﬁes how to group together tuples when computing the window function. Use **PARTITION BY** to specify group.

```SQL
SELECT cid, sid, ROW_NUMBER() OVER (PARTITION BY cid)
FROM enrolled ORDER BY cid;
```

![0054.jpg](assets/1677724457982-63d66807-07a8-4360-9b96-432ea2acd60c.jpeg)

We can also put an **ORDER BY** within OVER to ensure a deterministic ordering of results even if database changes internally.

```SQL
SELECT *, ROW_NUMBER() OVER (ORDER BY cid)
FROM enrolled ORDER BY cid;
```

![0055.jpg](assets/1677724457807-4825ff8a-dc31-4dd4-b928-645449955de3.jpeg)

**IMPORTANT:**  The DBMS computes RANK after the window function sorting, whereas it computes ROW_NUMBER before the sorting.
Example: Find the student with the second highest grade for each course.

```SQL
SELECT * FROM (
	SELECT *, RANK() OVER (PARTITION BY cid
		ORDER BY grade ASC) AS rank
	FROM enrolled) AS ranking
WHERE ranking.rank = 2;
```

![0056.jpg](assets/1677724459327-5a7a7da9-c0ec-4955-b784-ee32ac94e53b.jpeg)

# Common Table Expressions

Common Table Expressions (CTEs) are an alternative to windows or nested queries when writing more complex queries. They provide a way to write auxiliary statements for user in a larger query. CTEs can be thought of as a **temporary table** that is scoped to a single query.
The **WITH **clause binds the output of the inner query to a temporary result with that name.
Example: *Generate a CTE called cteName that contains a single tuple with a single attribute set to “1”. Select all attributes from this CTE. cteName.*

```SQL
WITH cteName AS (
SELECT 1
)
SELECT * FROM cteName;
```

![0057.jpg](assets/1677724459328-2f3fc5cc-8749-428c-ae44-108de0239551.jpeg)

We can bind output columns to names before the AS:

```SQL
WITH cteName (col1, col2) AS (
SELECT 1, 2
)
SELECT col1 + col2 FROM cteName;
```

A single query may contain multiple CTE declarations:

```SQL
WITH cte1 (col1) AS (SELECT 1), cte2 (col2) AS (SELECT 2)
SELECT * FROM cte1, cte2;
```

![0058.jpg](assets/1677724461423-17f79494-2a0e-44d8-b5e0-8c2afafca54e.jpeg)

![0059.jpg](assets/1677724461278-ed40b79a-d3e6-446b-bb46-ff3eb9e1f4e5.jpeg)

![0060.jpg](assets/1677724461417-abd1af3f-d8cd-4b56-b97f-eb010cd91193.jpeg)

Adding the **RECURSIVE** keyword after WITH allows a CTE to reference itself. This enables the implementation of recursion in SQL queries. With recursive CTEs, SQL is provably Turing-complete, implying that it is as computationally expressive as more general purpose programming languages (if a bit more cumbersome).
Example: *Print the sequence of numbers from 1 to 10.*

```SQL
WITH RECURSIVE cteSource (counter) AS (
  ( SELECT 1 )
  UNION
  ( SELECT counter + 1 FROM cteSource
  WHERE counter < 10 )
)
SELECT * FROM cteSource;
```

![0061.jpg](assets/1677724462744-1d0247b5-6e4d-42c7-ae14-57e179034ffb.jpeg)

![0062.jpg](assets/1677724462237-3f8dfc34-ea5b-4859-bad1-10468c9db4df.jpeg)

![0063.jpg](assets/1677724464685-c2de8634-cd8d-428a-a255-c22648278c44.jpeg)

![0064.jpg](assets/1677724464331-e1240a9f-b048-43e0-b543-6fb4f937c9a2.jpeg)
