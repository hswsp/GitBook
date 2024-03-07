# 14 - Query Planning & Optimization

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183440302-a1943c44-c8a4-486c-b3dd-6b1a27801785.jpeg#averageHue=%231e2836&clientId=u2c114338-f416-4&from=ui&id=u834a3193&name=1.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=606649&status=done&style=none&taskId=u55021dea-b8b6-4647-969c-eb5cfd200ee&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183439633-865a43be-24e5-40ae-802b-d7007335e41d.jpeg#averageHue=%23eeeded&clientId=u2c114338-f416-4&from=ui&id=u665481e9&name=2.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=282420&status=done&style=none&taskId=uf2f3bd57-cb93-4651-9bbc-f865b5f2dd7&title=)

# Overview

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183439792-61e70495-6197-48c3-9b8a-a0ca5d37f4d6.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=ud2b0a0af&name=3.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=357343&status=done&style=none&taskId=u8ce5666a-5b84-4b5f-806d-9f62f640ca7&title=)

Because SQL is declarative, the query only tells the DBMS what to compute, but not how to compute it. Thus, the DBMS needs to translate a SQL statement into an executable query plan. But there are different ways to execute each operator in a query plan (e.g., join algorithms) and there will be differences in performance among these plans. The job of the DBMS’s optimizer is to pick an optimal plan for any given query.

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183439837-b410e3ab-3bb2-4596-a203-25227f87c014.jpeg#averageHue=%23eaeaea&clientId=u2c114338-f416-4&from=ui&id=ud0e00605&name=4.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=365277&status=done&style=none&taskId=u11cc4f4d-1004-423b-a484-b0fcb096571&title=)

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183439919-a7c58d6e-9268-4e44-b0ae-c936cea9c8b4.jpeg#averageHue=%23dedede&clientId=u2c114338-f416-4&from=ui&id=u32f82735&name=5.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=true&size=394800&status=done&style=none&taskId=u502960f2-bdca-4be6-bbd9-84f2f857c1b&title=Figure%201%3A%20Architecture%20Overview%20%E2%80%93%20The%20application%20connected%20to%20the%20database%20system%20and%20sends%20a%20SQL%20query%2C%20which%20may%20be%20rewritten%20to%20a%20different%20format.%20The%20SQL%20string%20is%20parsed%20into%20tokens%20that%20make%20up%20the%20syntax%20tree.%20The%20binder%20converts%20named%20objects%20in%20the%20syntax%20tree%20to%20internal%20identi%EF%AC%81ers%20by%20consulting%20the%20system%20catalog.%20The%20binder%20emits%20a%20logical%20plan%20which%20may%20be%20fed%20to%20a%20tree%20rewriter%20for%20additional%20schema%20info.%20The%20logical%20plan%20is%20given%20to%20the%20optimizer%20which%20selects%20the%20most%20ef%EF%AC%81cient%20procedure%20to%20execute%20the%20plan.)
**Figure 1: Architecture Overview** – The application connected to the database system and sends a SQL query, which may be rewritten to a different format. The SQL string is parsed into tokens that make up the syntax tree. The binder converts named objects in the syntax tree to internal identiﬁers by consulting the system catalog. The binder emits a logical plan which may be fed to a tree rewriter for additional schema info. The logical plan is given to the optimizer which selects the most efﬁcient procedure to execute the plan.

The ﬁrst implementation of a query optimizer was IBM System R and was designed in the 1970s. Prior to this, people did not believe that a DBMS could ever construct a query plan better than a human. Many concepts and design decisions from the System R optimizer are still in use today.
There are two high-level strategies for query optimization.
The ﬁrst approach is to use static rules, or ***heuristics***. Heuristics match portions of the query with known patterns to assemble a plan. These rules transform the query to remove inefﬁciencies. Although these rules may **require consultation of the catalog to understand the structure of the data, they never need to examine the data itself.** 
An alternative approach is to use ***cost-based search*** to read the data and estimate the cost of executing equivalent plans. The cost model chooses the plan with the lowest cost.
Query optimization is the most difﬁcult part of building a DBMS. Some systems have attempted to apply **machine learning** to improve the accuracy and efﬁciency of optimizers, but no major DBMS currently deploys an optimizer based on this technique.

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183442467-cab3223d-1e30-4ab8-9b37-c5f118ba6a4b.jpeg#averageHue=%23ececec&clientId=u2c114338-f416-4&from=ui&id=u630b9219&name=6.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=318345&status=done&style=none&taskId=uf8974834-4276-4972-8d81-5e95d7f0945&title=)

## Logical vs. Physical Plans

The optimizer generates a mapping of a ***logical algebra expression*** to the optimal equivalent physical algebra expression. The logical plan is roughly equivalent to the relational algebra expressions in the query.

**Physical operators**  deﬁne a speciﬁc execution strategy **using an access path** for the different operators in the query plan. Physical plans may depend on the physical format of the data that is processed (i.e. sorting, compression).

There does not always exist a one-to-one mapping from logical to physical plans.

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183441990-03d08de1-e1b1-435a-a99f-e0123b17ec7a.jpeg#averageHue=%23efefef&clientId=u2c114338-f416-4&from=ui&id=ud55c7acb&name=7.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=194343&status=done&style=none&taskId=u3e4ce390-64e0-4d15-b282-5a6cee72605&title=)

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183442683-5ffa1630-176e-450a-b5c4-3e030f0c1445.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=ubc1ddefe&name=8.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=331030&status=done&style=none&taskId=u144a2cfb-50bd-4e0b-a4f0-c3d5ff424d1&title=)

# Logical Query Optimization

Some selection optimizations include:

- Perform ﬁlters as early as possible (predicate pushdown).
- **Reorder predicates** so that the DBMS applies the most selective one ﬁrst.
- Breakup a complex predicate and pushing it down (split conjunctive predicates).

An example of predicate pushdown is shown in ??.

Some projection optimizations include:

- Perform projections as early as possible to create smaller tuples and reduce intermediate results (*projection pushdown*).
- Project out all attributes except the ones requested or requires.

An example of projection pushdown in shown in Figure 2.

![Screen Shot 2023-04-13 at 10.47.54 AM.png](assets/Screen+Shot+2023-04-13+at+10.47.54+AM-20231123131905-vki5sab.png)

**Figure 2: Projection Pushdown** – Since the query only asks for the student name and ID, the DBMS can remove all columns except for those two before applying the join.

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183442624-d9dc8c01-bfab-49b4-b757-092b6df42205.jpeg#averageHue=%23eeeeee&clientId=u2c114338-f416-4&from=ui&id=ubc2ad086&name=9.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=231550&status=done&style=none&taskId=uc9e675c5-293b-4ac2-a6a2-0c63a55dfdc&title=)

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183442905-df40da88-9700-4b80-84d9-afb023214a58.jpeg#averageHue=%23e0e0e0&clientId=u2c114338-f416-4&from=ui&id=u577a40a9&name=10.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=353046&status=done&style=none&taskId=u90283fb6-2822-4be8-84a6-095affdf59d&title=)

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183444379-95bdbd82-441c-4770-b119-e9eba2cfdc78.jpeg#averageHue=%23e1e1e1&clientId=u2c114338-f416-4&from=ui&id=u1b28a6ad&name=11.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=322031&status=done&style=none&taskId=u519f1a80-8ef0-4a82-8e19-0073e5bbff2&title=)

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183446386-fb217b6e-6d13-44db-93d1-92a2e0624ecd.jpeg#averageHue=%23e0e0e0&clientId=u2c114338-f416-4&from=ui&id=u6803362c&name=12.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=337877&status=done&style=none&taskId=u78e5acc2-2349-4c89-9e17-cb8f3f152e4&title=)

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183446401-b601bdf5-420a-4c1e-a682-846f0fe5077c.jpeg#averageHue=%23e1e0e0&clientId=u2c114338-f416-4&from=ui&id=ua275511c&name=13.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=343623&status=done&style=none&taskId=uf02cef15-2fb5-4444-b62e-4d412969d91&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183446605-ec13a5fd-9cc4-431f-9827-b15dc7c64d62.jpeg#averageHue=%23e0e0e0&clientId=u2c114338-f416-4&from=ui&id=u9d015e52&name=14.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=371744&status=done&style=none&taskId=u94701449-fecc-4089-a0cc-0068bc90c45&title=)

The DBMS can also optimize nested sub-queries without referencing a cost model. There are two different approaches to this type of optimization:

• Re-write the query by de-correlating and / or ﬂattening it. An example of this is shown in Figure 6.

• Decompose the nested query and store the result to a temporary table. An example of this is shown in Figure 7.

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183446528-752e4cb3-cd3f-4122-baee-e2842e7f7642.jpeg#averageHue=%23ececec&clientId=u2c114338-f416-4&from=ui&id=u4dec97eb&name=15.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=297760&status=done&style=none&taskId=u29824f43-0572-4229-b439-25ae1881db6&title=)

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183447094-546f5ed5-ee22-4e6f-95d2-cfec2e112594.jpeg#averageHue=%23e4e4e4&clientId=u2c114338-f416-4&from=ui&id=ub8f24086&name=16.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=296471&status=done&style=none&taskId=uaadbadcc-4d39-4354-bdb0-f01adb274e1&title=)

**Figure 6: Subquery Optimization** - Rewriting The former query can be rewritten as the latter query by rewriting the subquery as a `JOIN`. Removing a level of nesting in this way effectively *ﬂattens* the query.

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183448437-c55628f3-dfb3-424c-9b4f-abc10ed42b4a.jpeg#averageHue=%23ededed&clientId=u2c114338-f416-4&from=ui&id=u7d2a697e&name=17.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=274616&status=done&style=none&taskId=ue856d6d2-3bf4-4885-9dcc-35f403d3e8d&title=)

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183448781-b42256b2-31b0-4c25-8210-3ae8adb799e4.jpeg#averageHue=%23e4e4e4&clientId=u2c114338-f416-4&from=ui&id=ue7ca18b6&name=18.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=303146&status=done&style=none&taskId=u785ea1e2-8fe2-4c0d-a907-9cb9881a509&title=)

**Figure 7: Subquery Optimization - Decomposition** – For complex queries with subqueries, the DBMS optimizer may break up the original query into blocks and focus on optimizing each individual block at a a time. In this example, the optimizer decomposes a query with a nested aggregation by pulling the nested query out into its own query, and subsequently using this result to realize the logic of the original query.

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183449005-851e9763-8085-478c-b5a4-2cc0d9a9571d.jpeg#averageHue=%23e2e2e2&clientId=u2c114338-f416-4&from=ui&id=u7502baf7&name=19.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=298630&status=done&style=none&taskId=u48d30e0a-a32d-4213-afa9-570daacf40b&title=)

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183449147-e795e0d4-ae76-4d22-86cc-ceb4da0abc7c.jpeg#averageHue=%23e3e3e3&clientId=u2c114338-f416-4&from=ui&id=u6e8f80b7&name=20.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=316622&status=done&style=none&taskId=u65bc7191-b92d-431f-b391-3e2a833fc73&title=)

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183449252-4907ee30-9e83-4274-8d18-eb367ec94b8d.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=u0847c735&name=21.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=349017&status=done&style=none&taskId=uaa9e545f-9927-49b8-bd7b-5eba2a3c123&title=)

Another optimization that a DBMS can use is to remove impossible or *unnecessary predicates*. In this optimization, the DBMS elides evaluation of predicates whose result does not change per tuple in a table. Bypassing these predicates reduces computation cost. Figure 3 shows two examples of *unnecessary predicates*.

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183450440-8be8297d-208f-468d-aad0-bb58084f66cf.jpeg#averageHue=%23ededed&clientId=u2c114338-f416-4&from=ui&id=u5ac99902&name=22.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=198013&status=done&style=none&taskId=ua36a67ee-8f13-44de-8fd8-002575f429a&title=)

**Figure 3: Unnecessary Predicates** – The predicate in the ﬁrst query will always be false and can be disregarded. The former query can be rewritten as the latter query to produce the same result but save on computation.

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183451016-359b7b2b-76a8-4c20-951a-760f9fc0b6c3.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=u1be55492&name=23.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=237680&status=done&style=none&taskId=ud269f674-5027-4340-a7a2-666781dd7cb&title=)

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183451188-731aec95-65d4-4858-bea6-725483ad2933.jpeg#averageHue=%23e9e9e9&clientId=u2c114338-f416-4&from=ui&id=u33512db3&name=24.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=271177&status=done&style=none&taskId=u13e95369-0606-4967-803e-73e36c3a4ef&title=)

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183451560-7b697037-4cb6-469e-a39e-2b671726f35c.jpeg#averageHue=%23e4e4e4&clientId=u2c114338-f416-4&from=ui&id=u98e9b1e2&name=25.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=356644&status=done&style=none&taskId=u285c0209-12cc-45c1-b3a4-dedb8b6f6d4&title=)

A similar optimization is merging predicates. An example of this optimization is shown in Figure 4.

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183451588-9760b9b6-9f6b-484b-a354-1aea315e54a3.jpeg#averageHue=%23e5e5e5&clientId=u2c114338-f416-4&from=ui&id=u06457091&name=26.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=329368&status=done&style=none&taskId=ucf358c0c-65c0-4aa2-a947-01aa0efd138&title=)

**Figure 4: Merging Predicates** – The WHERE predicate in query 1 has redundancy as what it is searching for is any value between 1 and 150. Query 2 shows the more succinct way to express request in query 1.

The ordering of `JOIN` operations is a key determinant of query performance. Exhaustive enumeration of all possible join orders is inefﬁcient, so join-ordering optimization requires a cost model. However, we can still eliminate *unnecessary joins* with a heuristic approach to optimization. An example of join elimination is shown in Figure 5.

![Screen Shot 2023-04-13 at 11.20.23 AM.png](assets/Screen+Shot+2023-04-13+at+11.20.23+AM-20231123131905-1vlx6y7.png)

**Figure 5: Join Elimination** – The join in query 1 is wasteful because every tuple in A must exist in A. Query 1 can instead be written as query 2.

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183452474-17c91786-1c59-40e9-b222-b8580d919331.jpeg#averageHue=%23ecebeb&clientId=u2c114338-f416-4&from=ui&id=u70160490&name=27.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=342633&status=done&style=none&taskId=uec9c855c-00dd-4751-9495-5960a563384&title=)

# Cost Estimations

DBMS’s use cost models to estimate the cost of executing a plan. These models evaluate equivalent plans for a query to help the DBMS select the most optimal one.

The cost of a query depends on several underlying metrics, including:

• **CPU**: small cost, but tough to estimate.

• **Disk I/O**: the number of block transfers.

• **Memory**: the amount of DRAM used.

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183453658-cddfb983-7af2-4584-90ff-4ee7aa163101.jpeg#averageHue=%23ececec&clientId=u2c114338-f416-4&from=ui&id=u2dd78d83&name=28.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=326899&status=done&style=none&taskId=u1c13eb31-171e-4b3b-bd2c-f69f78d5da5&title=)

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183454157-933fbab8-bb96-4196-9eab-1210dd7508cc.jpeg#averageHue=%23eaeaea&clientId=u2c114338-f416-4&from=ui&id=u79df0c7f&name=29.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=365823&status=done&style=none&taskId=uc80c0783-30c7-45ed-9e57-5cde308efc5&title=)

Exhaustive enumeration of all valid plans for a query is much too slow for an optimizer to perform. For joins alone, which are commutative and associative, there are $4^n$ different orderings of every n-way join. Optimizers must limit their search space in order to work efﬁciently.

To approximate costs of queries, DBMS’s maintain internal *statistics* about tables, attributes, and indexes in their internal catalogs. Different systems maintain these statistics in different ways. Most systems attempt to avoid on-the-ﬂy computation by maintaining an internal table of statistics. These internal tables may then be updated in the background.

For each relation $R$, the DBMS maintains the following information:

- $N_R$: Number of tuples in R
- $V (A, R)$: Number of distinct values of attribute A

With the information listed above, the optimizer can derive the selection *cardinality* $SC(A, R)$ statistic. The selection cardinality is the average number of records with a value for an attribute $A$ given $\frac{N_R}{V (A,R)}$ . Note that this assumes data uniformity. This assumption is often incorrect, but it simpliﬁes the optimization process.

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183454361-91be85a9-2738-4bbd-bef8-7fa00165b64e.jpeg#averageHue=%23ececec&clientId=u2c114338-f416-4&from=ui&id=ue95bab45&name=30.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=324445&status=done&style=none&taskId=u3b8727d9-3050-4f77-8442-9382b3486ec&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183455163-6008e2c8-cc39-471a-a73e-79a7d68b2d6d.jpeg#averageHue=%23eeeeee&clientId=u2c114338-f416-4&from=ui&id=uae4cf8a7&name=31.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=498501&status=done&style=none&taskId=uf0d2793e-836a-47e9-99d5-e4239e76f1b&title=)

## Selection Statistics

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183455384-86f5606e-a263-4868-b472-034ddbd73552.jpeg#averageHue=%23ededed&clientId=u2c114338-f416-4&from=ui&id=uc8db5f1b&name=32.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=318835&status=done&style=none&taskId=u9f70d429-570c-45cc-94a0-059927d75b3&title=)

The **selection cardinality** can be used to determine the number of tuples that will be selected for a given input.

Equality predicates on unique keys are simple to estimate (see Figure 8). A more complex predicate is shown in Figure 9.

![Screen Shot 2023-04-13 at 11.40.40 AM.png](assets/Screen+Shot+2023-04-13+at+11.40.40+AM-20231123131905-sdrd4z2.png)

**Figure 8: Simple Predicate Example** – In this example, determining what index to use is easy because the query contains an equality predicate on a unique key.

![Screen Shot 2023-04-13 at 11.40.45 AM.png](assets/Screen+Shot+2023-04-13+at+11.40.45+AM-20231123131905-nrw29ac.png)

**Figure 9: Complex Predicate Example** – More complex predicates, such as range or conjunctions, are harder to estimate because the selection cardinalities of the predicates must be combined in non-trivial ways.

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183455556-9fe6457e-7811-46d6-9f7b-570d94ccea38.jpeg#averageHue=%23efefef&clientId=u2c114338-f416-4&from=ui&id=u7a9ad26e&name=33.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=198983&status=done&style=none&taskId=uc0202224-598b-4c1a-8884-e45ebcdcd0e&title=)

The *selectivity* (sel) of a predicate P is the fraction of tuples that qualify. The formula used to compute selective depends on the type of predicate. Selectivity for complex predicates is hard to estimate accurately which can pose a problem for certain systems. An example of a selectivity computation is shown in Figure 10.

![Screen Shot 2023-04-13 at 11.48.36 AM.png](assets/Screen+Shot+2023-04-13+at+11.48.36+AM-20231123131905-iauxvl4.png)

**Figure 10: Selectivity of Negation Query Example** – The selectivity of the negation query is computed by subtracting the selectivity of the positive query from 1. In the example, the answer comes out to be $\frac{4}{5}$ which is accurate.

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183458320-327ef883-a3ef-4a04-b3d2-12f2dedfaa74.jpeg#averageHue=%23eaeae9&clientId=u2c114338-f416-4&from=ui&id=u592a22eb&name=34.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=409073&status=done&style=none&taskId=ub0838b1d-07cb-412d-8821-988d226c299&title=)

Observe that the **selectivity** of a predicate is equivalent to the probability of that predicate. This allows probability rules to be applied in many selectivity computations. This is particularly useful when dealing with complex predicates. For example, if we assume that multiple predicates involved in a conjunction are *independent*, we can compute the total selectivity of the conjunction as the product of the selectivities of the individual predicates.

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183458649-27891351-3861-40eb-985a-61aa46a0d00c.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=u947b7b0f&name=35.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=342639&status=done&style=none&taskId=u5908052b-c3c1-4a47-a3ba-7d2a96344bc&title=)

## Selectivity Computation Assumptions

In computing the selection cardinality of predicates, the following three assumptions are used.

- **Uniform Data**: The distribution of values (except for the heavy hitters) is the same.
- **Independent Predicates**: The predicates on attributes are independent.
- **Inclusion Principle:**  The domain of join keys overlap such that each key in the inner relation will also exist in the outer table.

These assumptions are often not satisﬁed by real data. For example, *correlated attributes* break the assumption of independence of predicates.

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183458768-20732e26-0833-422f-bf28-ee5d49600fe0.jpeg#averageHue=%23ecebeb&clientId=u2c114338-f416-4&from=ui&id=u8061a7dc&name=36.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=342607&status=done&style=none&taskId=u6af81a20-be0f-4fb2-8920-289bbdb0ac2&title=)

# Histograms

![37.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183458707-a25bac9c-5ec3-4526-b571-8e4b9cf3dc00.jpeg#averageHue=%23ececec&clientId=u2c114338-f416-4&from=ui&id=u988b36a1&name=37.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=322601&status=done&style=none&taskId=uf6e0cb85-c5be-4c87-beaf-3a98736f238&title=)

![38.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183458779-9bb1d2c2-734b-49a5-a5b0-1dd5abe29890.jpeg#averageHue=%23ededed&clientId=u2c114338-f416-4&from=ui&id=u51d97046&name=38.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=323088&status=done&style=none&taskId=u99da2168-fdf1-46dc-8b78-30ba169c08a&title=)

Real data is often skewed and is tricky to make assumptions about. However, storing every single value of a data set is expensive. One way to reduce the amount of memory used by storing data in a *histogram* to group together values. An example of a graph with buckets is shown in Figure 11.

![39.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183460698-90be2441-23a1-4213-852d-e6049a5c39b9.jpeg#averageHue=%23edecec&clientId=u2c114338-f416-4&from=ui&id=u8e625831&name=39.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=312555&status=done&style=none&taskId=u0c62f4b4-e7e8-4811-a3de-aa415da50a4&title=)

![40.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183461331-32f2ee57-5814-4be1-b4a4-17ca0a7a7e96.jpeg#averageHue=%23edecec&clientId=u2c114338-f416-4&from=ui&id=uc19016d1&name=40.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=349383&status=done&style=none&taskId=u8657796d-e5f7-47e8-bcb4-19ff56f24ff&title=)

**Figure 11: Equi-Width Histogram**: The ﬁrst ﬁgure shows the original frequency count of the entire data set. The second ﬁgure is an equi-width histogram that combines together the counts for adjacent keys to reduce the storage overhead.

![41.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183461454-2b2f665e-0173-4758-ba5c-8768f7e216de.jpeg#averageHue=%23edecec&clientId=u2c114338-f416-4&from=ui&id=uc488be43&name=41.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=344999&status=done&style=none&taskId=u160943cd-8aa8-4d09-911a-64afad47d3f&title=)

Another approach is to use a equi-depth histogram that varies the width of buckets so that the **total number of occurrences for each bucket is roughly the same**. An example is shown in Figure 12.

![42.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183461428-7506a169-fd2c-4ad6-8a3a-2fa47357ae2d.jpeg#averageHue=%23ededed&clientId=u2c114338-f416-4&from=ui&id=u9a066cb2&name=42.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=309715&status=done&style=none&taskId=u3256d2a4-7b28-4cd9-8f36-d44f68807d8&title=)

![43.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183461443-4a1c1104-9ec8-4499-8ff2-6e51539cb5d9.jpeg#averageHue=%23ededed&clientId=u2c114338-f416-4&from=ui&id=u58ced95f&name=43.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=264509&status=done&style=none&taskId=u79fe3c4d-b7c1-4c15-8ee5-0b0ec0a90a8&title=)

**Figure 12: Equi-Depth Histogram** – To ensure that each bucket has roughly the same number of counts, the histogram varies the range of each bucket.

In place of histograms, some systems may use *sketches* to generate approximate statistics about a data set.

![44.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183462918-f1741dbe-7893-4a58-af45-cdbc40b4d457.jpeg#averageHue=%23ececec&clientId=u2c114338-f416-4&from=ui&id=ub3ce48f2&name=44.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=361857&status=done&style=none&taskId=uf374db27-6ca6-4d93-9c25-35d2a77cf08&title=)

# Sampling

DBMS’s can use *sampling* to apply predicates to a smaller copy of the table with a similar distribution (see Figure 13). The DBMS updates the sample whenever the amount of changes to the underlying table exceeds some threshold (e.g., 10% of the tuples).

![45.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183464083-0e33099c-7d7e-4fcb-a4a6-94dadbde4f4c.jpeg#averageHue=%23e3e3e3&clientId=u2c114338-f416-4&from=ui&id=ub8346276&name=45.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=458492&status=done&style=none&taskId=uce1f094d-433d-494e-8eb7-c27d57c5514&title=)

**Figure 13: Sampling** – Instead of using one billion values in the table to estimate selectivity, the DBMS can derive the selectivities for predicates from a subset of the original table.

![46.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183463746-885e68b7-3c7c-4b01-80fc-c26d7b3783b8.jpeg#averageHue=%23eeeeee&clientId=u2c114338-f416-4&from=ui&id=ue44539e2&name=46.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=223906&status=done&style=none&taskId=u5dff0f0b-ab38-408b-8e02-8a661bcf8da&title=)

![47.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183463985-94fd1da7-7584-4f56-964b-d0834c31be2c.jpeg#averageHue=%23ececec&clientId=u2c114338-f416-4&from=ui&id=u74cbc7f6&name=47.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=313412&status=done&style=none&taskId=uc0f755a5-bc2f-4581-a102-a95d231416b&title=)

# Single-Relation Query Plans

For single-relation query plans, the biggest obstacle is choosing the best access method (i.e., sequential scan, binary search, index scan, etc.) Most new database systems just use heuristics, instead of a sophisticated cost model, to pick an access method.

For OLTP queries, this is especially easy because they are sargable (Search Argument Able), which means that there exists a best index that can be selected for the query. This can also be implemented with simple heuristics.

![48.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183463991-fb0b70f4-b313-408f-9ac3-3ed897df50f0.jpeg#averageHue=%23ececec&clientId=u2c114338-f416-4&from=ui&id=u0928d966&name=48.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=284526&status=done&style=none&taskId=ua6c0b73a-ba64-4996-8581-998a038fb48&title=)

![49.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183465470-0ded6b73-3c10-4f43-a0b4-bab7d1c523ca.jpeg#averageHue=%23e8e7e7&clientId=u2c114338-f416-4&from=ui&id=u9978ad5c&name=49.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=383627&status=done&style=none&taskId=u66c3f662-4f07-4576-8e73-a6347223cc4&title=)

# Multi-Relation Query Plans

For Multi-Relation query plans, as number of joins increases, the number of alternative plans grow rapidly. Consequently, it is important to restrict the search space so as to be able to ﬁnd the optimal plan in a reasonable amount of time. There are two ways to approach this search problem:

- **Bottom-up**: Start with nothing and then build up the plan to get to the outcome that you want.
  Examples: IBM System R, DB2, MySQL, Postgres, most open-source DBMSs.
- **Top-down**: Start with the outcome that you want, and then work down the tree to ﬁnd the optimal plan that gets you to that goal. Examples: MSSQL, Greenplum, CockroachDB, Volcano

![50.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183465722-93384f21-9277-420a-99b6-405dee8a1661.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=uac6d5ef0&name=50.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=309819&status=done&style=none&taskId=u6e7148c6-17a4-4801-bdbd-1ae31880c76&title=)

## Bottom-up optimization example - System R

Use static rules to perform initial optimization. Then use dynamic programming to determine the best join order for tables using a divide-and conquer search method.

- Break query up into blocks and generate the logical operators for each block.
- For each logical operator, generate a set of physical operators that implement it.
- Then, iteratively construct a ”left-deep” tree that minimizes the estimated amount of work to execute the plan

![51.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183466285-7626a27c-fefc-4a28-97a2-fbd961bb2add.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=ubfebb401&name=51.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=311693&status=done&style=none&taskId=u5285b090-41f8-454e-bdc8-eefc7dc0376&title=)

![52.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183467331-7d2fbdb0-d19d-40e8-b154-778b93012202.jpeg#averageHue=%23e8e8e8&clientId=u2c114338-f416-4&from=ui&id=u5d616e9f&name=52.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=406901&status=done&style=none&taskId=u0f46b11a-de80-4383-a787-2b971e56f58&title=)

![53.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183466873-45e16d78-f64b-4c25-8034-6c89ec6a12c3.jpeg#averageHue=%23e7e7e7&clientId=u2c114338-f416-4&from=ui&id=u2085a8e7&name=53.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=418133&status=done&style=none&taskId=u0712f766-d9ac-4aad-a4fe-b085ea5fe9d&title=)

![54.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183467599-3012e82d-3591-4c3b-81e9-205ad3b5830e.jpeg#averageHue=%23e7e7e7&clientId=u2c114338-f416-4&from=ui&id=uba9f6b1b&name=54.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=315555&status=done&style=none&taskId=u4abfab18-a82b-4257-80ef-3fd8e8cfbc1&title=)

![55.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183467810-939b3e02-b21e-4c3e-a8ec-2c6b5b2c7d87.jpeg#averageHue=%23eaeaea&clientId=u2c114338-f416-4&from=ui&id=u7b0f0b49&name=55.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=274512&status=done&style=none&taskId=u81a0fb84-1a25-4db8-ae59-4c70b869814&title=)

![56.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183468635-caf18b37-d2d2-4cf2-bddb-21a53e48ae5c.jpeg#averageHue=%23e3e2e2&clientId=u2c114338-f416-4&from=ui&id=u97bd99eb&name=56.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=408348&status=done&style=none&taskId=ua115847d-c903-455d-a715-a8359e944cb&title=)

![57.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183469348-e12e987d-ee78-4bae-a6b0-5bff3f797da0.jpeg#averageHue=%23e7e6e6&clientId=u2c114338-f416-4&from=ui&id=u7d475e11&name=57.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=355991&status=done&style=none&taskId=u390422b3-c99e-4e67-a4a9-85848b82009&title=)

![58.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183469368-1b3d5e3f-545a-4cf7-ab14-92a6bb1d1956.jpeg#averageHue=%23edecec&clientId=u2c114338-f416-4&from=ui&id=u749d901e&name=58.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=222152&status=done&style=none&taskId=u2a698974-733d-4fd3-baf8-46d0651de4e&title=)

![59.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183469727-eaf6458a-7a30-4a0d-a439-d9d91be3dd8b.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=u56878de4&name=59.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=290057&status=done&style=none&taskId=u0f6672fa-9eb5-42a7-8be6-1b369b66140&title=)

## Top-down optimization example - Volcano

Start with a logical plan of what we want the query to be. Perform a branch-and-bound search to traverse the plan tree by converting logical operators into physical operators.

- Keep track of global best plan during search.
- Treat physical properties of data as ﬁrst-class entities during planning.

![60.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183470122-ba4bd572-0ba5-4c1b-9342-fe58a624e82e.jpeg#averageHue=%23e6e6e6&clientId=u2c114338-f416-4&from=ui&id=u1bc90445&name=60.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=412926&status=done&style=none&taskId=uf8761589-5a77-42b5-96fc-681716fc07a&title=)

![61.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183471107-7c0573df-ff5d-4761-b7a4-a8c0629ee3ee.jpeg#averageHue=%23ebebeb&clientId=u2c114338-f416-4&from=ui&id=u2f72143c&name=61.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=301425&status=done&style=none&taskId=u06fdfbb3-d4a3-4200-81c8-820f0eca983&title=)

![62.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183472160-4eeaa764-1525-434f-9158-7b41606353d8.jpeg#averageHue=%23e9e9e9&clientId=u2c114338-f416-4&from=ui&id=u8fd74b48&name=62.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=317257&status=done&style=none&taskId=u27167048-36e2-4ecf-a8fb-d158a1eda3d&title=)

![63.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183472494-31053fd8-b654-4ada-a84e-533fab166663.jpeg#averageHue=%23e9e9e9&clientId=u2c114338-f416-4&from=ui&id=u4015902f&name=63.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=323865&status=done&style=none&taskId=u301e2f8d-e24f-4658-b58d-9ef7ed97536&title=)

![64.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183472982-94f08aec-3cab-4622-ba39-39ccb479e042.jpeg#averageHue=%23e8e8e8&clientId=u2c114338-f416-4&from=ui&id=uad1ca256&name=64.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=335243&status=done&style=none&taskId=u3a3c5a19-2f10-4e90-ad4c-3e903154a27&title=)

![65.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183473361-bbd48655-5f64-4c04-9108-ecbc38ab4e75.jpeg#averageHue=%23e8e7e7&clientId=u2c114338-f416-4&from=ui&id=u33000a56&name=65.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=346491&status=done&style=none&taskId=uf167d22e-42b4-4bb8-a19f-01d42fccbaf&title=)

![66.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183473572-10a5bc4f-6bd2-4742-b380-fab2e7b79fae.jpeg#averageHue=%23e7e6e6&clientId=u2c114338-f416-4&from=ui&id=u25ed777c&name=66.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=358137&status=done&style=none&taskId=uf841760a-2837-4c6b-bf46-609cffc5a6a&title=)

![67.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183474361-4d21cc14-5f46-4b8c-80e0-441957db7e1e.jpeg#averageHue=%23e7e6e6&clientId=u2c114338-f416-4&from=ui&id=u6a25cbb0&name=67.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=365846&status=done&style=none&taskId=u477a35f8-431b-4dc2-89d4-c048d8865a3&title=)

![68.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183474593-bdd5a2b6-9be2-4a19-b7f2-ac0d4b5149e9.jpeg#averageHue=%23e5e5e5&clientId=u2c114338-f416-4&from=ui&id=u94ca95ee&name=68.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=412199&status=done&style=none&taskId=u569642a3-afde-48e3-a934-da3dbd5ae67&title=)

![69.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183475440-94944f01-0fd9-40b5-9139-74d11e907626.jpeg#averageHue=%23e4e3e3&clientId=u2c114338-f416-4&from=ui&id=u70ba3aa2&name=69.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=425864&status=done&style=none&taskId=u5dc70c5e-e712-44f6-a2ec-f0109ef0bdb&title=)

![70.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183476129-4c520ebb-f45f-4b68-b630-97431aa8845d.jpeg#averageHue=%23e4e4e3&clientId=u2c114338-f416-4&from=ui&id=ub61cb1b5&name=70.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=428687&status=done&style=none&taskId=u15908351-0864-4665-aaa7-73f132e233c&title=)

![71.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183476274-a5a56ea3-ac17-41b2-aefc-cc8f430f6c74.jpeg#averageHue=%23e2e2e2&clientId=u2c114338-f416-4&from=ui&id=uc9dc74a3&name=71.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=440047&status=done&style=none&taskId=u8ab94d9c-e579-4a43-a7d3-774481eee34&title=)

![72.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183476925-88c4dfe6-fb8b-47b0-8e4b-186c0123bceb.jpeg#averageHue=%23e2e2e2&clientId=u2c114338-f416-4&from=ui&id=ucfcf822c&name=72.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=440009&status=done&style=none&taskId=u51c6696d-02b8-489e-a44a-8db0baea518&title=)

![73.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183476951-588836f3-e90d-441d-81df-126f02970c82.jpeg#averageHue=%23e1e0e0&clientId=u2c114338-f416-4&from=ui&id=u849ee131&name=73.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=454740&status=done&style=none&taskId=ubc9d8ad3-90b3-4709-98b9-fa1b0533d23&title=)

![74.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183478676-b8e5b3c1-5635-4db0-8763-260ea053af4d.jpeg#averageHue=%23e1e1e1&clientId=u2c114338-f416-4&from=ui&id=u302f2f20&name=74.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=457329&status=done&style=none&taskId=ucdfaa403-0a79-4197-9181-cdfbd4fe943&title=)

![75.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183478751-f8c7be89-c206-40de-95b8-c8310be0e903.jpeg#averageHue=%23eeeeee&clientId=u2c114338-f416-4&from=ui&id=u77331a3c&name=75.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=261158&status=done&style=none&taskId=u7f87b33b-820c-4da9-9387-b1313f1b490&title=)

![76.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1681183478640-08a124d0-df3c-407b-9234-a7e61374dae5.jpeg#averageHue=%23f0f0f0&clientId=u2c114338-f416-4&from=ui&id=u300df27d&name=76.jpg&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=170685&status=done&style=none&taskId=u2230c905-8e4d-4510-8caa-4e7626b7637&title=)
