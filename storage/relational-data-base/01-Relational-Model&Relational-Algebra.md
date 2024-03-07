# 01 - Relational Model & Relational Algebra

![01-introduction\_1.png](assets/20240307110106.png)

![01-introduction\_2.png](assets/20240307110213.png)

![01-introduction\_3.png](assets/20240307110228.png)

![01-introduction\_4.png](assets/20240307110324.png)

![01-introduction\_5.png](assets/20240307110350.png)

![01-introduction\_6.png](assets/20240307110507.png)

![01-introduction\_7.png](assets/20240307110527.png)

![01-introduction\_8.png](assets/20240307110536.png)

![01-introduction\_9.png](assets/20240307110545.png)

![01-introduction\_10.png](assets/1677688006255-218aec46-8740-4de1-ae83-a50f32c934f2.png)

![01-introduction\_11.png](assets/1677688006786-4f8ee136-05da-4cf0-9259-d71bc259ec2a.png)

![01-introduction\_12.png](assets/1677688019844-a8f45b2d-4fd8-4328-80df-45a3216f8137.png)

![01-introduction\_13.png](assets/1677688010872-54e12184-26eb-4edd-95f2-2f44bb857dcd.png)

## DataBase

A database is an organized collection of inter-related data that models some aspect of the real-world (e.g., modeling the students in a class or a digital music store). People often confuse “databases” with “database management systems” (e.g., MySQL, Oracle, MongoDB, Snowflake). A database management system (DBMS) is the software that manages a database.

![01-introduction\_14.png](assets/1677688011527-1d17f089-c90b-4255-9219-db150a60dba3.png)

Consider a database that models a digital music store (e.g., Spotify). Let the database hold information about the artists and which albums those artists have released.

![01-introduction\_15.png](assets/1677688014874-fbc3c123-cf61-4931-af66-fdf8b5d270b3.png)

## Flat File Strawman

Database is stored as comma-separated value (CSV) files that the DBMS manages. Each entity will be stored in its own file. The application has to parse files each time it wants to read or update records.

![01-introduction\_16.png](assets/1677688015480-b4d0f69c-b96b-43a5-8ede-1f6bad1f5fdc.png)

Keeping along with the digital music store example, there would be two files: one for artist and the other for album. Each entity has its own set of attributes, so in each file, different records are delimited by new lines, while each of the corresponding attributes within a record are delimited by a comma. E.g.: An artist could have a name, year, and country attributes, while an album has name, artist and year attributes. Below is an example CSV file for information about artists with the schema (name, year, country):

![01-introduction\_17.png](assets/1677688018989-d7d491a8-751b-467b-a473-c4aec9d90ef5.png)

![01-introduction\_18.png](assets/1677688018989-36a50929-1bcd-4897-ad88-e52475163702.png)

### Issues with Flat File

#### Data Integrity

* How do we ensure that the artist is the same for each album entry?
* What if somebody overwrites the album year with an invalid string?
* How do we treat multiple artists on one album?
* What happens when we delete an artist with an album?

![01-introduction\_19.png](assets/1677688020215-b91b681f-cb45-4a82-b85c-7648d94919b4.png)

#### Implementation

* How do we find a particular record?
* What if we now want to create a new application that uses the same database?
* What if two threads try to write to the same file at the same time?

![01-introduction\_20.png](assets/1677688023116-fe0e0a27-11c4-4781-84d6-d51816f5e2f1.png)

#### Durability

* What if the machine crashes while our program is updating a record?
* What if we want to replicate the database on multiple machines for high availability?

![01-introduction\_21.png](assets/1677688023123-fbf9b8cc-1e43-49b1-89e9-04da67ebaa22.png)

## Database Management System

A DBMS is a software that allows applications to store and analyze information in a database. A general-purpose DBMS is designed to allow the definition, creation, querying, update, and administration of databases in accordance with some data model.

![01-introduction\_22.png](assets/1677688025203-39369f72-523d-4bea-aaee-14d7debaaa60.png)

A data model is a collection of concepts for describing the data in database.

* Examples: relational (most common), NoSQL (key/value, graph), array/matrix/vectors

**A schema is a description of a particular collection of data based on a data model.**

![01-introduction\_24.png](assets/1677688027147-c1cc2aef-b9b3-4175-a323-3facf2086924.png)

### Early DBMSs

Database applications were difficult to build and maintain because there was a tight coupling between logical and physical layers.

The logical layer describes which entities and attributes the database has while the physical layer is how those entities and attributes are being stored. Early on, the physical layer was defined in the application code, so if we wanted to change the physical layer the application was using, we would have to change all of the code to match the new physical layer.

![01-introduction\_25.png](assets/1677688028523-1d54bb1d-a0a6-481d-a60b-54dd6673aab8.png)

## Relational Model

Ted Codd noticed that people were rewriting DBMSs every time they wanted to change the physical layer, so in 1970 he proposed the relational model to avoid this.

![01-introduction\_27.png](assets/1677688054049-f7951c4b-2f9b-44ee-b2a3-5ce818ecc9bc.png)

The relational model defines a database abstraction based on relations to avoid maintenance overhead. It has three key points:

* Store database in simple data structures (relations).
* Access data through high-level language, **DBMS figures out best execution strategy**.
* Physical storage left up to the DBMS implementation.

![01-introduction\_28.png](assets/1677688031287-fed85144-3523-4c5a-af92-98d07be0c268.png)

The relational data model defines three concepts:

* **Structure**: The definition of relations and their contents. This is the attributes the relations have and the values that those attributes can hold.
* **Integrity**: Ensure the database’s contents satisfy constraints. An example constraint would be that any value for the year attribute has to be a number.
* **Manipulation**: How to access and modify a database’s contents.

![01-introduction\_29.png](assets/1677688031690-78cb480a-720a-448a-9e15-1a9ba2a65bd0.png)

**A \_relation**\*\*\* is an unordered set\*\* that contains the relationship of attributes that represent entities. Since the relationships are unordered, the DBMS can store them in any way it wants, allowing for optimization. \*\*A \*\*\*\*\*tuple\*\*\*\*​ **\_** is a set of attribute values \*\*(also known as its _domain_) in the relation. Originally, values had to be atomic or scalar, but **now values can also be lists or nested data structures**. Every attribute can be a special value, NULL, which means for a given tuple the attribute is undefined. A relation with n attributes is called an _**n-ary relation.**_

![01-introduction\_30.png](assets/1677688034737-c84e5e75-3bbb-4320-af39-2dea359369ae.png)

### Keys

A relation’s primary key uniquely identifies a single tuple. Some DBMSs automatically create an internal primary key if you do not define one. A lot of DBMSs have support for autogenerated keys so an application does not have to manually increment the keys, but a primary key is still required for some DBMSs.

![01-introduction\_31.png](assets/1677688037215-81270916-af37-4d3c-871c-e9a4cc555a5b.png)

![01-introduction\_32.png](assets/1677688037505-94228ff2-2fd8-43ec-9a34-c5bbac00dd9b.png)

A _foreign key_ specifies that an attribute from one relation has to map to a tuple in another relation.

![01-introduction\_33.png](assets/1677688035410-d145ea46-2e1e-41f2-bfe0-569eb8d8eaaa.png)

![01-introduction\_34.png](assets/1677688039360-2a09ad05-19b9-44e5-bc6d-209becbabd2f.png)

## Data Manipulation Languages (DMLs)

Methods to store and retrieve information from a database. There are two classes of languages for this:

* **Procedural**: The query specifies the (high-level) strategy the DBMS should use to find the desired result based on sets / bags. (relational algebra)
* **Non-Procedural (Declarative)** : The query specifies only what data is wanted and not how to find it.(relational calculus)

![01-introduction\_35.png](assets/1677688040742-ad673938-8396-4dbd-8045-ba56a37c7dec.png)

## Relational Algebra

_Relational Algebra_ is a set of fundamental operations to retrieve and manipulate tuples in a relation. Each operator takes in one or more relations as inputs, and outputs a new relation. To write queries we can “chain” these operators together to create more complex operations.

![01-introduction\_36.png](assets/1677688042989-efecd576-36f9-4ff2-9473-9ea63f19dfa2.png)

### Select

Select takes in a relation and outputs a subset of the tuples from that relation that satisfy a selection predicate. The predicate acts like a filter, and we can combine multiple predicates using conjunctions and disjunctions. Syntax: $σ\_{predicate} (R)$. Example: $σ\_{a\_id}=’a2’ (R)$ SQL: `SELECT * FROM R WHERE a_id = 'a2'`

![01-introduction\_37.png](assets/1677688043115-4529391b-d400-4eb7-8d47-9355111e1a93.png)

### Projection

Projection takes in a relation and outputs a relation with tuples that contain only specified attributes. You can rearrange the ordering of the attributes in the input relation as well as manipulate the values. Syntax: $π\_{A1,A2,. . . ,An} (R)$. Example: $π\_{b\_id-100, a\_id}(σ\_{a\_id}=’a2’ (R))$ SQL: `SELECT b_id-100, a_id FROM R WHERE a_id = 'a2'`

![01-introduction\_38.png](assets/1677688045120-10628d8e-8901-4d49-83e3-638a3c983378.png)

### Union

Union takes in two relations and outputs a relation that **contains all tuples** that appear in at least one of the input relations. Note: The two input relations have to have the exact same attributes. Syntax: $(R ∪ S)$. SQL: `(SELECT * FROM R) UNION ALL (SELECT * FROM S)`

![01-introduction\_39.png](assets/1677688045782-e5e2df67-0b30-45f4-bd95-e1e981b5d402.png)

## Intersection

Intersection takes in two relations and outputs a relation that contains all tuples that appear in **both** of the input relations. Note: The two input relations have to have the exact same attributes. Syntax: $(R ∩ S)$. SQL: `(SELECT * FROM R) INTERSECT (SELECT * FROM S)`

![01-introduction\_40.png](assets/1677688047768-7b2a7e92-9840-4876-a360-995f03ee3a8f.png)

### Difference

Difference takes in two relations and outputs a relation that contains all tuples that appear in the \*\*first relation but not the second \*\*relation. Note: The two input relations have to have the exact same attributes. Syntax: $(R − S)$. SQL: `(SELECT * FROM R) EXCEPT (SELECT * FROM S)`

![01-introduction\_41.png](assets/1677688047816-928b88d1-a762-4d33-abd1-0948ae6cf1a5.png)

### Product

Product takes in two relations and outputs a relation that contains **all possible combinations** for tuples from the input relations. Syntax: $(R × S)$. SQL: `(SELECT * FROM R) CROSS JOIN (SELECT * FROM S)`, or simply `SELECT * FROM R, S`

![01-introduction\_42.png](assets/20240307112043.png)

### Join

Join takes in two relations and outputs a relation that contains all the tuples that are a **combination of two tuples** where for **each attribute that the two relations share,** the values for that attribute of both tuples is the same. Syntax: $(R ▷◁ S)$. SQL: `SELECT * FROM R JOIN S USING (ATTRIBUTE1, ATTRIBUTE2...)`

![01-introduction\_43.png](assets/1677688051285-5e0e68d2-1866-4dfe-b7db-a157312ceade.png)

![01-introduction\_44.png](assets/1677688051782-c5a8c1b8-b859-40a0-975e-0c4622f63121.png)

## Observation

**Relational algebra is a procedural language** because it defines the high level-steps of how to compute a query. For example, $σ\_{b\_id=102} (R ▷◁ S)$ is saying to first do the join of R and S and then do the select, whereas ($R ▷◁ (σ\_{b\_id=102} (S))$) will do the select on S first, and then do the join. **These two statements will actually produce the same answer**, but if there is only 1 tuple in S with $b\_id=102$ out of a billion tuples, then ($R ▷◁ (σ\_{b\_id=102} (S))$) will be significantly faster than $σ\_{b\_id=102} (R ▷◁ S)$.

A better approach is to say the result you want (retrieve the joined tuples from R and S where $b\_id$ equals 102), and **let the DBMS decide the steps it wants to take to compute the query.** SQL will do exactly this, and it is the de facto standard for writing queries on relational model databases.

![01-introduction\_45.png](assets/1677688052641-7ac958d6-725f-46d5-a6c1-1c9e3a9285bb.png)

![01-introduction\_46.png](assets/1677688056824-4652afce-9a75-4e27-9162-22efae07f164.png)

![01-introduction\_47.png](assets/1677688056826-2adacad5-5f74-4be8-9a62-ee679085390e.png)

![01-introduction\_48.png](assets/1677688055427-3336b038-3fc3-4902-8e6b-e547cb71827c.png)

![01-introduction\_49.png](assets/1677688056015-0ab156ed-eb75-4089-82f5-fbe2107c4b81.png)

![01-introduction\_50.png](assets/1677688060145-611a66eb-ab93-4e10-9dfe-c709591ad217.png)

![01-introduction\_51.png](assets/1677688060159-f488d1de-1eb4-4baa-9784-8db70b5a4bb5.png)

![01-introduction\_52.png](assets/1677688058805-96f34449-834a-47b0-b840-716345edb976.png)
