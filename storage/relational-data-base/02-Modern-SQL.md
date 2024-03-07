# 02 - Modern SQL

![0001.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724420254-796ab31f-c5cd-4618-af7c-bc242a32dbbc.jpeg#averageHue=%231d2635&clientId=u8e0e4d99-7235-4&from=ui&id=ub575a445&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=342605&status=done&style=none&taskId=u09821f7f-7762-4c6a-bdf8-7659d42062c&title=)

![0002.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724419262-5d7a224f-0b12-4726-bf5e-60c15b9a8bc4.jpeg#averageHue=%23ececec&clientId=u8e0e4d99-7235-4&from=ui&id=ucaf07264&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=211838&status=done&style=none&taskId=u82a5ed90-ed9e-451e-80d1-6c4df62407f&title=)

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

![0003.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724419724-2a3215c5-e610-446a-8a3e-64b8cd606161.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=uf1b1e589&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=248950&status=done&style=none&taskId=u8862376c-7da5-49ad-a489-cc6913a8352&title=)

![0004.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724420411-8f8218c4-2125-4d94-9c69-baf59d0acfc7.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u5f2f00b3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=347051&status=done&style=none&taskId=ufad766b7-88b5-4857-b990-57d43113df9&title=)

SQL is not a dead language. It is being updated with new features every couple of years. SQL-92 is the minimum that a DBMS has to support to claim they support SQL. Each vendor follows the standard to a certain degree but there are many proprietary extensions.

![0005.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724420175-e3ea27e8-237a-474e-860c-56051b14e753.jpeg#averageHue=%23eae9e9&clientId=u8e0e4d99-7235-4&from=ui&id=uec2a2f81&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=285803&status=done&style=none&taskId=ud712825a-b193-414d-9cc5-cc032cbc2ec&title=)

Some of the major updates released with each new edition of the SQL standard are shown below.

- SQL:1999 Regular expressions, Triggers
- SQL:2003 XML, Windows, Sequences
- SQL:2008 Truncation, Fancy sorting
- SQL:2011 Temporal DBs, Pipelined DML
- SQL:2016 JSON, Polymorphic tables

![0006.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724424080-39024196-4b10-47f2-91c3-959abee768b2.jpeg#averageHue=%23dfdfdf&clientId=u8e0e4d99-7235-4&from=ui&id=u70be83a3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=405400&status=done&style=none&taskId=uf5add8a9-05c4-4f6a-9e60-7b4498392a7&title=)

![0007.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724422979-07eb45aa-18ea-4fcd-92ef-76ab26d070bd.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u6a485c01&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=241580&status=done&style=none&taskId=u621569f5-c1ac-4eae-bf63-14f01839f6d&title=)

![0008.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724422730-619c9772-ed77-49b0-91cf-3e825a1823a9.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=ua50040ec&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=194537&status=done&style=none&taskId=u2cd73c5c-537a-4af0-a83d-654a062d3ca&title=)

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

![0009.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724424020-51e12686-baf9-442f-9c26-b80069a76549.jpeg#averageHue=%23d8d7d7&clientId=u8e0e4d99-7235-4&from=ui&id=u38940a75&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=321137&status=done&style=none&taskId=ud16c9834-f33f-4d2e-8fbb-449ede436b3&title=)

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

![0010.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724424065-0f0a4357-0ffc-4d1d-b5ba-7da36b4b417b.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=u8c799ab8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=225517&status=done&style=none&taskId=u87663f4b-7be6-4ec3-b9cb-cb56a1fcfa0&title=)

Example: *Get # of students with a ‘@cs’ login*.
The following three queries are equivalent:

```SQL
SELECT COUNT(*) FROM student WHERE login LIKE '%@cs';

SELECT COUNT(login) FROM student WHERE login LIKE '%@cs';

SELECT COUNT(1) FROM student WHERE login LIKE '%@cs';
```

![0011.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724426726-8c4e625a-bb7e-4744-9b53-a86bf4af7169.jpeg#averageHue=%23ebebea&clientId=u8e0e4d99-7235-4&from=ui&id=u2df899c2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=279299&status=done&style=none&taskId=u7263a23c-648f-44c1-b520-f48bb5e8054&title=)

**A single SELECT statement can contain multiple aggregates**:
Example: *Get # of students and their average GPA with a ‘@cs’ login*.

```SQL
SELECT AVG(gpa), COUNT(sid) FROM student WHERE login LIKE '%@cs';
```

![0012.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724426476-f4f546aa-f54a-4b86-b43e-89853faf71aa.jpeg#averageHue=%23eaeaea&clientId=u8e0e4d99-7235-4&from=ui&id=uafb6b0c7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=219006&status=done&style=none&taskId=u1bce359e-8606-45b4-b845-586b9a79f37&title=)

Some aggregate functions (e.g. COUNT, SUM, AVG) support the DISTINCT keyword:
Example: *Get # of unique students and their average GPA with a ‘@cs’ login.*

```SQL
SELECT COUNT(DISTINCT login) FROM student WHERE login LIKE '%@cs';
```

![0013.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724427478-2a05f988-6ace-4ddc-85c0-fa3c58fab6b3.jpeg#averageHue=%23eaeaea&clientId=u8e0e4d99-7235-4&from=ui&id=u8ae267c7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=230353&status=done&style=none&taskId=u4b128b31-9c06-44a4-8624-067e90433cd&title=)

Output of other columns outside of an aggregate is **undeﬁned** (e.cid is undeﬁned below).
Example: *Get the average GPA of students in each course.*

```SQL
SELECT AVG(s.gpa), e.cid FROM enrolled AS e, student AS s WHERE e.sid = s.sid;
```

![0014.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724427718-01d0dc32-1f9e-4961-b63f-3318df99d4f8.jpeg#averageHue=%23ececec&clientId=u8e0e4d99-7235-4&from=ui&id=ufe3a3723&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=224585&status=done&style=none&taskId=ua707b793-9ba7-40e8-be54-fe30c9a2b8b&title=)

![0015.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724427981-896ae9da-bcd1-4663-bced-9f3d0feb384e.jpeg#averageHue=%23eaeaea&clientId=u8e0e4d99-7235-4&from=ui&id=uc33c24fa&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=236762&status=done&style=none&taskId=u5cf4b83b-ffe0-40e9-8149-3ed5ec601f3&title=)

Non-aggregated values in SELECT output clause must appear in GROUP BY clause.

```SQL
SELECT AVG(s.gpa), e.cid 
	FROM enrolled AS e, student AS s
WHERE e.sid = s.sid
	GROUP BY e.cid;
```

![0016.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724430542-b1719d69-7071-4c70-bc8f-21d452506661.jpeg#averageHue=%23e2e0e0&clientId=u8e0e4d99-7235-4&from=ui&id=u7700685a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=317755&status=done&style=none&taskId=u5b62e433-3997-48cb-9a91-fa0fd746729&title=)

![0017.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724430156-7be416bf-cc9b-4733-a073-98990b29fa6d.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=u9ad21c72&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=217476&status=done&style=none&taskId=ufaf3bd7e-6938-4ddc-a912-cc5a6530b5a&title=)

![0018.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724431209-e1688f6f-de21-4017-a3fa-5aea994afc89.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=u530a60cb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=219448&status=done&style=none&taskId=u9bdc8846-9a29-4882-a14f-75a75c3cb57&title=)

![0019.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724431171-73a12877-4620-41d0-a285-db7b77724c51.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=ud13e58dc&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=221858&status=done&style=none&taskId=u4c726ce7-c826-463d-8f95-0f7c48e658e&title=)

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

![0020.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724431704-da20dab9-b865-4f39-94a3-b089299c7453.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=u218aeec8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=227736&status=done&style=none&taskId=uba017965-e293-4a31-b1d3-1aae859638b&title=)

![0021.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724433635-5ed34174-9f4a-4b09-9312-62ff95a85a2b.jpeg#averageHue=%23edecec&clientId=u8e0e4d99-7235-4&from=ui&id=ubd704992&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=239676&status=done&style=none&taskId=u84f58be2-4aa8-4388-995a-6f8955d9cac&title=)

![0022.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724433954-55a9f4c7-f86e-4009-b113-7b639c7751cf.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=u653a0758&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=234381&status=done&style=none&taskId=u797ddc89-4358-4886-b69b-b073285d7b8&title=)

![0023.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724435137-7d703594-d78a-48b1-80d4-5dd6548c1c60.jpeg#averageHue=%23e6e5e5&clientId=u8e0e4d99-7235-4&from=ui&id=uda5f1c45&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=278482&status=done&style=none&taskId=u4281ad3f-b551-4ce8-94df-bd705f96314&title=)

# String Operations

The SQL standard says that strings are **case sensitive **and **single-quotes only**. There are functions to manipulate strings that can be used in any part of a query.

![0024.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724436180-c547d54c-68d9-43fa-9cff-990ab6ef01da.jpeg#averageHue=%23eae9e9&clientId=u8e0e4d99-7235-4&from=ui&id=u64af7e87&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=280188&status=done&style=none&taskId=u22845e8b-4a7e-4d96-a18f-491c7d4d8b0&title=)

**Pattern Matching: **The LIKE keyword is used for string matching in predicates.

- “%” matches any substrings (including empty).
- “_” matches any one character.

![0025.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724436173-eb4ad929-28e7-40c2-8e95-a3060a38e7c8.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=uc6e75592&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=253796&status=done&style=none&taskId=u79063014-336a-4dc4-995b-8092010ecfa&title=)

**String Functions SQL-92** deﬁnes string functions. Many database systems implement other functions in addition to those in the standard. Examples of standard string functions include `SUBSTRING(S, B, E)` and `UPPER(S)`.

![0026.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724437528-2b643692-2c4f-4a78-b7fe-af0554f939b6.jpeg#averageHue=%23eaeaea&clientId=u8e0e4d99-7235-4&from=ui&id=u864dde7f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=267690&status=done&style=none&taskId=udd00ca45-325d-45da-a1dd-162d7ac9b4e&title=)

**Concatenation:**  Two vertical bars (“||”) will concatenate two or more strings together into a single string.

![0027.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724437736-23e8e069-e239-40bc-886b-71acd9bf9db1.jpeg#averageHue=%23e9e9e9&clientId=u8e0e4d99-7235-4&from=ui&id=ud105903e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=298305&status=done&style=none&taskId=ue49c7574-6027-453a-92ce-4d075a5db00&title=)

# Date and Time

Operations to manipulate DATE and TIME attributes. Can be used in either output or predicates. The speciﬁc syntax for date and time operations varies wildly across systems.

![0028.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724438581-c13a6005-b201-4cb4-a073-aa702066c0fb.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u0f313e26&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=216867&status=done&style=none&taskId=ufa7d8190-a30b-4ac0-9aa1-4e5ad0b51c6&title=)

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

![0029.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724440116-e53ab8bc-78b9-44b8-a0e7-b518e5c5d754.jpeg#averageHue=%23eaeaea&clientId=u8e0e4d99-7235-4&from=ui&id=u4596573f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=262668&status=done&style=none&taskId=uc2266c1d-d400-4c34-a2a0-25d16a44d27&title=)

![0030.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724440132-55184289-c45d-4485-8528-a9189079abff.jpeg#averageHue=%23eaeae9&clientId=u8e0e4d99-7235-4&from=ui&id=u2dcc4fd2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=276141&status=done&style=none&taskId=uab372832-01b0-4c4a-aa4b-f744aa8d99a&title=)

![0031.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724441698-833e154d-d1da-4ed5-85e0-2d50ce0c9ba7.jpeg#averageHue=%23ebeaea&clientId=u8e0e4d99-7235-4&from=ui&id=ua91d6d7e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=252259&status=done&style=none&taskId=u707eee23-c83e-492d-a08e-4d6b5005fa1&title=)

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

![0032.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724441710-461eb6b2-ffe5-47ba-a157-9e47bc61af8a.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u2c277121&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=230995&status=done&style=none&taskId=u3d72bc30-a80a-4c88-8123-befa599948f&title=)

We can use multiple ORDER BY clauses to break ties or do more complex sorting:

```SQL
SELECT sid, grade FROM enrolled WHERE cid = '15-721'
ORDER BY grade DESC, sid ASC;
```

![0033.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724442687-a68da143-5dcb-474a-a80b-e3a0667f9752.jpeg#averageHue=%23eaeae9&clientId=u8e0e4d99-7235-4&from=ui&id=ufdfa32c0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=292572&status=done&style=none&taskId=ucd69b64c-07ce-47a2-909c-e7270febbb5&title=)

We can also use any arbitrary expression in the ORDER BY clause:

```SQL
SELECT sid FROM enrolled WHERE cid = '15-721'
	ORDER BY UPPER(grade) DESC, sid + 1 ASC;
```

![0034.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724444221-99df1a66-93de-4933-8e2c-e9537fb2e16d.jpeg#averageHue=%23ebeaea&clientId=u8e0e4d99-7235-4&from=ui&id=u5242f3aa&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=295981&status=done&style=none&taskId=u5e18bbaf-e929-4cd1-9237-6b6fc8c4fc7&title=)

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

![0035.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724443900-2d759ba8-5e91-41f2-a43c-e4f8f60b9fa3.jpeg#averageHue=%23ececec&clientId=u8e0e4d99-7235-4&from=ui&id=u84d94a22&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=248444&status=done&style=none&taskId=u02f54080-f894-4d35-9108-6504f3d3d1c&title=)

![0036.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724445639-3983a837-d6f7-466c-9e1e-2ac9ac377b00.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=ub55eff90&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=281139&status=done&style=none&taskId=u972ea770-0b33-4ad7-88d3-31702ce51c2&title=)

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

![0037.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724445219-7d6f764c-72f9-4bf8-b0cc-980a7fd986bc.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u310472ab&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=237447&status=done&style=none&taskId=ued66428e-405c-499b-b68b-7867a0e2c38&title=)

Example: *Get the names of students that are enrolled in ‘15-445’.*

```SQL
SELECT name FROM student
WHERE sid IN (
SELECT sid FROM enrolled
WHERE cid = '15-445'
);
```

Note that *sid* has different scope depending on where it appears in the query.

![0038.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724445743-b316840b-49e3-47e4-8e13-ab063d608665.jpeg#averageHue=%23eeeeee&clientId=u8e0e4d99-7235-4&from=ui&id=u172abe6d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=189325&status=done&style=none&taskId=ue265eb65-9621-4ff2-ac85-863366785de&title=)

![0039.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724446992-0ff3019a-6394-460e-8117-39c957be3f00.jpeg#averageHue=%23eeeeee&clientId=u8e0e4d99-7235-4&from=ui&id=u2f87f083&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=189741&status=done&style=none&taskId=ua8075b29-7017-4d15-8d25-134e821b7ec&title=)

![0040.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724447507-6692ddae-43d2-4141-8563-0530caee2749.jpeg#averageHue=%23eeeded&clientId=u8e0e4d99-7235-4&from=ui&id=u90ba82d5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=197322&status=done&style=none&taskId=u5b35c934-6fc1-4ea0-8269-1847a80292c&title=)

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

![0041.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724448569-f3896532-d71e-4c2b-aa34-ff90adefe808.jpeg#averageHue=%23ececec&clientId=u8e0e4d99-7235-4&from=ui&id=u4b8bd105&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=231729&status=done&style=none&taskId=ue5d20754-c0a3-4c13-914e-8dda164e7e6&title=)

![0042.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724448530-0250a640-5a4e-4dfc-be37-d165f200855f.jpeg#averageHue=%23eeeeee&clientId=u8e0e4d99-7235-4&from=ui&id=u654f2ce4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=193457&status=done&style=none&taskId=u73b3ae01-295f-4449-ace2-6c76b20ebab&title=)

![0043.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724449823-ac5178fe-fb06-4718-a1fe-0efd821fdea1.jpeg#averageHue=%23eaeaea&clientId=u8e0e4d99-7235-4&from=ui&id=ub6ceb410&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=264782&status=done&style=none&taskId=u4cb1dd93-fd1f-4491-bc9c-2e8424487ad&title=)

![0044.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724450104-557b734a-6a45-474e-9296-dffba9191055.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=uf417d1a1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=203150&status=done&style=none&taskId=u84554d7c-80ab-4d6a-b912-9424aef7018&title=)

![0045.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724451556-1a5d80cd-de37-4278-a97b-13e46a368b15.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=u3cf443f8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=208915&status=done&style=none&taskId=u1d211379-3fa0-4227-b2c3-d80c2009755&title=)

![0046.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724451699-84c7630d-74fa-4d9a-a90e-0b5f61ecb6cc.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u4bc102aa&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=215795&status=done&style=none&taskId=ua2fc06f1-804b-45ce-9041-ff10d5f80dc&title=)

![0047.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724452103-c96ee36e-5e16-4ae5-b6b0-61c574fd774c.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=uae707082&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=236002&status=done&style=none&taskId=u98cef5e2-1b0e-4ed7-8139-d5b5ae09aa3&title=)

![0048.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724453834-66186baa-f267-4599-afeb-60aaab6d4ea7.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=uda2a22a5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=278127&status=done&style=none&taskId=u998ed41b-2bed-4b80-98eb-ef80b515269&title=)

![0049.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724453864-b7a5c4b0-aab3-4538-b919-6e8efd3c885a.jpeg#averageHue=%23e3e3e3&clientId=u8e0e4d99-7235-4&from=ui&id=u31bca901&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=280058&status=done&style=none&taskId=uc4bae235-a844-43fd-8169-f1f6a1f6624&title=)

![0050.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724454362-7f051dc1-72b5-462c-a2df-ec57a3cac38c.jpeg#averageHue=%23eeeeee&clientId=u8e0e4d99-7235-4&from=ui&id=u0a14a90a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=195066&status=done&style=none&taskId=u5322015e-a31e-4d7d-9073-5b2d4fe8ffc&title=)

![0051.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724455390-38793a85-e8e2-4646-851e-a38bbdf71946.jpeg#averageHue=%23e8e8e8&clientId=u8e0e4d99-7235-4&from=ui&id=u50f1e3ae&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=235154&status=done&style=none&taskId=u81008060-c442-4d90-aeed-b44fea8ae75&title=)

# Window Functions

A window function perform “sliding” calculation across a set of tuples that are related. Like an aggregation but tuples are not grouped into a single output tuple.
**Functions: **The window function can be any of the aggregation functions that we discussed above. There are also also special window functions:

1. `ROW_NUMBER`: The number of the current row.
2. `RANK`: The order position of the current row.

![0052.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724455887-489441fd-ff8d-4be8-9ead-115329bca42e.jpeg#averageHue=%23ecebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u3ba0bc28&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=271557&status=done&style=none&taskId=u56e0a876-2ae8-48c2-a382-473f7ac5d01&title=)

![0053.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724457838-cb5d6975-dcb7-4ac5-8e96-78116636e837.jpeg#averageHue=%23e5e4e4&clientId=u8e0e4d99-7235-4&from=ui&id=u581c773f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=307246&status=done&style=none&taskId=u12337258-049c-4395-93ec-c33a28b0746&title=)

**Grouping:**  The OVER clause speciﬁes how to group together tuples when computing the window function. Use **PARTITION BY** to specify group.

```SQL
SELECT cid, sid, ROW_NUMBER() OVER (PARTITION BY cid)
FROM enrolled ORDER BY cid;
```

![0054.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724457982-63d66807-07a8-4360-9b96-432ea2acd60c.jpeg#averageHue=%23e7e5e4&clientId=u8e0e4d99-7235-4&from=ui&id=ua8630550&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=302113&status=done&style=none&taskId=u95370548-02dd-4f99-bbd0-34814a30a79&title=)

We can also put an **ORDER BY** within OVER to ensure a deterministic ordering of results even if database changes internally.

```SQL
SELECT *, ROW_NUMBER() OVER (ORDER BY cid)
FROM enrolled ORDER BY cid;
```

![0055.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724457807-4825ff8a-dc31-4dd4-b928-645449955de3.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=ub592d161&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=215374&status=done&style=none&taskId=ud9ba4f69-8911-4c91-a811-db5d59f9d89&title=)

**IMPORTANT:**  The DBMS computes RANK after the window function sorting, whereas it computes ROW_NUMBER before the sorting.
Example: Find the student with the second highest grade for each course.

```SQL
SELECT * FROM (
	SELECT *, RANK() OVER (PARTITION BY cid
		ORDER BY grade ASC) AS rank
	FROM enrolled) AS ranking
WHERE ranking.rank = 2;
```

![0056.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724459327-5a7a7da9-c0ec-4955-b784-ee32ac94e53b.jpeg#averageHue=%23edecec&clientId=u8e0e4d99-7235-4&from=ui&id=u9dd7eb79&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=267020&status=done&style=none&taskId=ucd5ee09d-2ef9-48c7-98a1-dfb81429e63&title=)

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

![0057.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724459328-2f3fc5cc-8749-428c-ae44-108de0239551.jpeg#averageHue=%23ecebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u585caf18&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=235738&status=done&style=none&taskId=u4a7085ae-efd8-4e31-ba4b-0544becaa37&title=)

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

![0058.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724461423-17f79494-2a0e-44d8-b5e0-8c2afafca54e.jpeg#averageHue=%23ecebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u5f3a0a40&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=274897&status=done&style=none&taskId=uf78a68f9-9f1b-4104-b13a-d283af6de6c&title=)

![0059.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724461278-ed40b79a-d3e6-446b-bb46-ff3eb9e1f4e5.jpeg#averageHue=%23ececec&clientId=u8e0e4d99-7235-4&from=ui&id=uf3c03c33&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=256754&status=done&style=none&taskId=u20c08dbe-4f24-4592-8412-23822feb93c&title=)

![0060.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724461417-abd1af3f-d8cd-4b56-b97f-eb010cd91193.jpeg#averageHue=%23ebebeb&clientId=u8e0e4d99-7235-4&from=ui&id=u2383fb41&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=252334&status=done&style=none&taskId=ub189a8c6-11b0-4e65-9817-6676957f7c4&title=)

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

![0061.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724462744-1d0247b5-6e4d-42c7-ae14-57e179034ffb.jpeg#averageHue=%23ededed&clientId=u8e0e4d99-7235-4&from=ui&id=ub37b0976&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=234250&status=done&style=none&taskId=u04b600b5-c351-4386-a257-5e5f3ad2341&title=)

![0062.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724462237-3f8dfc34-ea5b-4859-bad1-10468c9db4df.jpeg#averageHue=%23eeeeee&clientId=u8e0e4d99-7235-4&from=ui&id=ubac8c240&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=173652&status=done&style=none&taskId=ud52df31a-958c-46d9-91e3-b09e686079e&title=)

![0063.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724464685-c2de8634-cd8d-428a-a255-c22648278c44.jpeg#averageHue=%23ececec&clientId=u8e0e4d99-7235-4&from=ui&id=u07aa4919&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=245602&status=done&style=none&taskId=u29a919d6-fd5f-4fa8-abf3-47cf6424476&title=)

![0064.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677724464331-e1240a9f-b048-43e0-b543-6fb4f937c9a2.jpeg#averageHue=%23f0f0f0&clientId=u8e0e4d99-7235-4&from=ui&id=u75911fee&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=132795&status=done&style=none&taskId=u9e929560-3abc-4ec1-9612-f1e3ff9719b&title=)
