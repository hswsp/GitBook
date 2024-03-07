# 24 - Embedded Database Logic

![1.jpg](assets/1-20231123131906-caszefb.jpg)

![2.jpg](assets/2-20231123131906-09iv835.jpg)

# Motivation

Until now, we have assumed that all of the logic for an application is located in the application itself. Most applications interact with the DBMS using a “conversational” API (e.g., JDBC, ODBC). This is where the application sends a query request to the DBMS and then waits for a response. After the DBMS sends a response, it then waits for the next request from the application for that connection.

It may be possible to move complex application logic into the DBMS to avoid multiple network round-trips. Doing this can improve efficiency, responsiveness, and reusability in the application.

The downside of these methods is that the syntax is often not portable across different DBMSs. And depending on the engineering practices of an organization, you may need to maintain different versions of the embedded database logic.

![3.jpg](assets/3-20231123131906-smoyf00.jpg)

![4.jpg](assets/4-20231123131906-re967ol.jpg)

![5.jpg](assets/5-20231123131906-kiu4lcg.jpg)

![6.jpg](assets/6-20231123131906-mhm4kge.jpg)

![7.jpg](assets/7-20231123131906-w0ydlsl.jpg)

![8.jpg](assets/8-20231123131906-epazojy.jpg)

![9.jpg](assets/9-20231123131906-qdtgpwg.jpg)

![10.jpg](assets/10-20231123131906-hs3vpzf.jpg)

# User-Defined Functions

A *user defined function* is a function written by the application developer that extends the system’s functionality beyond its built-in operations. Each function takes in scalar input arguments, performs some computation, and then returns a result (scalar, table). A UDF can only be invoked as part of a SQL statement.

![11.jpg](assets/11-20231123131906-hpqd7au.jpg)

Return Types:

• **Scalar Functions**: Return a single data value.

• **Table Functions**: Return a single result table.

Function Body:

• **SQL Functions:**  A SQL-based UDF contains a list of SQL statements that the DBMS executes in order when the UDF is invoked. The UDF returns whatever the result of the last query is.

• **Native Programming Language**: The developer can write a UDF in a language that is natively supported by the DBMS. Examples: *SQL/PSM* (SQL Standard), *PL/SQL* (Oracle, DB2), *PL/pgSQL* (Postgres), *Transact-SQL* (MSSQL/Sybase).

• **External Programming Language**: UDFs written in more conventional programming languages (e.g., C, Java, JavaScript, Python) run a separate process (i.e., sandbox) to prevent them from crashing the DBMS process.

![12.jpg](assets/12-20231123131906-ath3vqk.jpg)

![13.jpg](assets/13-20231123131906-i76e2bw.jpg)

![14.jpg](assets/14-20231123131906-f80bot0.jpg)

![15.jpg](assets/15-20231123131906-srsk4un.jpg)

![16.jpg](assets/16-20231123131906-voutqoe.jpg)

![17.jpg](assets/17-20231123131906-zqr2aok.jpg)

![18.jpg](assets/18-20231123131906-ublktcs.jpg)

![19.jpg](assets/19-20231123131906-0r4vise.jpg)

![20.jpg](assets/20-20231123131906-z89f6z2.jpg)

![21.jpg](assets/21-20231123131906-d0snn2b.jpg)

Using UDFs have some advantages:

• **Modularity and Code Reuse**: Different queries can reuse the same application logic without requiring reimplementation.

• **Reduced Network Overhead**: Queries will incur fewer network round-trips between the application server and DBMS for complex operations.

• **Readability**: Some types of application logic are easier to express and read as UDFs than SQL.

![22.jpg](assets/22-20231123131906-xmra0gd.jpg)

However, there are important pitfalls to be aware of when considering using a UDF:

• **Black Boxes**: UDFs are often treated as black boxes by query optimizers so you cannot estimate their cost.

• **Lack of Parallelism**: Parallelizing queries is challenging as a sequence of queries may be correlated. Some UDFs will incrementally construct the sequence of queries as they run. Some DBMSs will only use a single thread to execute queries with a UDF.

Complex UDFs executed iteratively without system optimizations can be very slow.

![23.jpg](assets/23-20231123131906-pzqi6zl.jpg)

![24.jpg](assets/24-20231123131906-r26hk3e.jpg)

![25.jpg](assets/25-20231123131906-qe6vh9o.jpg)

![26.jpg](assets/26-20231123131906-1zwrbqr.jpg)

# Stored Procedures

A *stored procedure* is a self-contained function that performs more complex logic inside of the DBMS. Unlike a UDF, a stored procedure can be invoked on its own without having to **be part of a SQL statement.**

UDFs are also usually meant to be read-only, while stored procedures are allowed to modify the DBMS.

![27.jpg](assets/27-20231123131906-dqpqy5h.jpg)

![28.jpg](assets/28-20231123131906-o5c13hq.jpg)

![29.jpg](assets/29-20231123131906-t22y8iu.jpg)

![30.jpg](assets/30-20231123131906-6awym1r.jpg)

![31.jpg](assets/31-20231123131906-rak88eh.jpg)

# Triggers

A *trigger* instructs the DBMS to invoke a UDF when some event occurs in the database. Some examples of trigger usage are constraint checking or auditing any time a tuple is modified in a table.

Each trigger is defined with the following properties:

• **Event Type**: Type of modification (`INSERT`, `UPDATE`, `DELETE`, `ALTER`).

• **Event Scope**: Scope of the modification (`TABLE`, `DATABASE`, `VIEW`).

• **Timing**: When the trigger should be activated based on statement (before, after, instead of).

![32.jpg](assets/32-20231123131906-o8j46z6.jpg)

![33.jpg](assets/33-20231123131906-39aonwh.jpg)

![34.jpg](assets/34-20231123131906-pfpwrq7.jpg)

![35.jpg](assets/35-20231123131906-mbs3nem.jpg)

![36.jpg](assets/36-20231123131906-2h3kcra.jpg)

# Change Notifications

A *change notification* is like a trigger except that the DBMS sends a message to an external entity that something notable has happened in the database. They can be chained with a trigger to pass along whenever a change occurs. Notifications are asynchronous, meaning that they are only pushed to listening connection whenever they interact with the DBMS. Some ORMs will poll the DBMS with lightweight “`SELECT 1`” every so often to retrieve new notifications.

Commands:

• `LISTEN`: The connection registers with the DBMS to listen for notifications at the named event queue.

• `NOTIFY`: Push a notification to any connection that is listening at named event queue. Syntax details vary per DBMS implementation.

![37.jpg](assets/37-20231123131906-dk43kw7.jpg)

![38.jpg](assets/38-20231123131906-m6c26vz.jpg)

![39.jpg](assets/39-20231123131906-4pskh3g.jpg)

![40.jpg](assets/40-20231123131906-rruky6j.jpg)

# User-Defined Types

Most DBMSs support the basic primitive types defined in the SQL standard (e.g., ints, floats, varchars). But sometimes that application wants to store complex types that are comprised of multiple primitive types. Or these complex types might have different behaviors for various arithmetic operators.

One potential solution is to store split the complex type and store each of its primitive element as its own attribute in the table. The problem with this is that you have to make sure that the application knows how to split/combine the complex type.

Another solution is to let the **application** **serialize** the complex type (e.g., Java “serialize”, Python “pickle”, Google Protobufs) and **store it as a blob in the database**. The problem with this approach is that it not possible to edit sub-attributes in the type without first deserializing the entire blob. Likewise, the DBMS’s optimizer is unable estimate selectivity on predicates that access serialized data.

![41.jpg](assets/41-20231123131906-h3ggdzl.jpg)

A better approach is to use a *user-defined type* (UDT). This is a special data type that is defined by the application developer that the DBMS can be stored natively. Each DBMS exposes a different API that allows you to create a UDT. This allows you override basic operators and functions.

![42.jpg](assets/42-20231123131906-2tf7iyj.jpg)

![43.jpg](assets/43-20231123131906-sixatls.jpg)

# Views

A database views is “virtual” table that contains the output from a `SELECT` query. The view can then be accessed as if it was a real table. Under the hood, queries on views are converted into a single query using the original query that generated view. Views allow programmers to simplify a complex query that is executed often. It is often also used as a mechanism for hiding a subset of a table’s attributes from certain users. One can only update a view if it only contains a **single** based table, and that it does not contain aggregations, distinctions, union, or grouping.

Unlike `SELECT...INTO`, a view does not allocate a table to store the result of the view. A *materialized* *view* maintains the result of a view internally that may be automatically updated when the underlying tables change.

![44.jpg](assets/44-20231123131906-s1wddvb.jpg)

![45.jpg](assets/45-20231123131906-wrfwdhi.jpg)

![46.jpg](assets/46-20231123131906-stpbmsm.jpg)

![47.jpg](assets/47-20231123131906-5uqex0x.jpg)

![48.jpg](assets/48-20231123131906-hz6g0hv.jpg)

![49.jpg](assets/49-20231123131906-q8vcloy.jpg)

![50.jpg](assets/50-20231123131906-h6pmzu0.jpg)
