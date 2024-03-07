# 12 - Query Execution 1

![1.jpg](assets/1680073295307-83e4dff6-a81f-4638-b5a4-763128ab9658.jpeg)

![2.jpg](assets/1680073294958-48396462-331b-4fbe-a677-3f9be165a3bb.jpeg)

![3.jpg](assets/1680073294793-6695aae1-e67e-4af0-8352-9a6660b0f5a7.jpeg)

[Defensive Programming](https://swc-osg-workshop.github.io/2017-05-17-JLAB/novice/python/05-defensive.html)
[How to profile a C program with Valgrind/Callgrind](https://medium.com/@jacksonbelizario/profiling-a-c-program-with-valgrind-callgrind-b41f15b31527)

![4.jpg](assets/1680073294547-df093fc1-9388-486e-b373-8199edaf4a79.jpeg)

# Query Plan

The DBMS converts a SQL statement into a query plan. Operators in the query plan are arranged in a tree. Data flows from the leaves of this tree towards the root. The output of the root node in the tree is the result of the query. Typically operators are binary (1–2 children). The same query plan can be executed in multiple ways.

# Processing Models

A DBMS *processing model* defines how the system executes a query plan. It specifies things like the direction in which the query plan is evaluated and what kind of data is passed between operators along the way. There are different models of processing models that have various trade-offs for different workloads.
These models can also be implemented to invoke the operators either from **top-to-bottom** or from **bottom-to-top**. Although the top-to-bottom approach is much more common, the bottom-to-top approach can allow for tighter control of caches/registers in pipelines.
The three execution models that we consider are:
• Iterator Model
• Materialization Model
• Vectorized / Batch Model

![5.jpg](assets/1680073294795-60a05d6e-2a12-4bed-8b31-7d7cd33c83c6.jpeg)

## Iterator Model

The *iterator model*, also known as the Volcano or Pipeline model, is the most common processing model and is used by almost every (row-based) DBMS.

![6.jpg](assets/1680073296305-79e6a508-4176-4515-a96d-403181b02bc2.jpeg)

The iterator model works by implementing a `Next` function for every operator in the database. Each node in the query plan calls `Next` on its children until the leaf nodes are reached, which start emitting tuples up to their parent nodes for processing. Each tuple is then processed up the plan as far as possible before the next tuple is retrieved. This is useful in disk-based systems because it allows us to fully use each tuple in memory before the next tuple or page is accessed. A sample diagram of the iterator model is shown in Figure 1.
Query plan operators in an iterator model are highly composible and easy to reason about because each operator can be implemented independent from their parent or child operators in the query plan tree so long as it implements a `Next` function as follows:
• On each call to `Next`, the operator returns either a single tuple or a null marker if there are no more tuples to emit.
• The operator implements a loop that calls `Next` on its children to retrieve their tuples and then process them. In this way, calling `Next` on a parent calls `Next` on its children. In response, the child node will return the next tuple that the parent must process.
The iterator model allows for *pipelining* where the DBMS can process a tuple through as many operators as possible before having to retrieve the next tuple. The series of tasks performed for a given tuple in the query plan is called a *pipeline*.
Some operators will block until children emit all of their tuples. Examples of such operators include joins, subqueries, and ordering (`ORDER BY`). Such operators are known as *pipeline breakers*.
Output control works easily with this approach (`LIMIT`) because an operator can stop invoking `Next` on its child (or children) operator(s) once it has all the tuples that it requires.

![7.jpg](assets/1680073296922-ca7c4c92-dfb5-45b3-869a-d2f2e487da65.jpeg)

**Figure 1: Iterator Model Example** – Pseudo code of the different `Next` functions for each of the operators. The `Next` functions are essentially for-loops that iterate over the output of their child operator. For example, the root node calls `Next` on its child, the join operator, which is an access method that loops over the relation R and emits a tuple up that is then operated on. After all tuples have been processed, a `null` pointer (or another indicator) is sent that lets the parent nodes know to move on.

![8.jpg](assets/1680073297044-8f7f58e9-2b37-4678-9026-b175992047e8.jpeg)

## Materialization Model

The *materialization* model is a specialization of the iterator model where each operator processes its input all at once and then emits its output all at once. Instead of having a `next` function that returns a single tuple, each operator returns all of its tuples every time it is reached. To avoid scanning too many tuples, the DBMS can propagate down information about how many tuples are needed to subsequent operators (e.g. `LIMIT`). The operator “materializes” its output as a single result. The output can be either a whole tuple (NSM) or a subset of columns (DSM). A diagram of the materialization model is shown in Figure 2.

![9.jpg](assets/1680073297156-2c40c503-f2cf-49ac-b3e1-40f11cda9eaa.jpeg)

![10.jpg](assets/1680073297650-79dbd930-d63b-4d96-9791-2dbc1cdf1611.jpeg)

Every query plan operator implements an `Output` function:
• The operator processes all the tuples from its children at once.
• The return result of this function is all the tuples that operator will ever emit. When the operator finishes executing, the DBMS never needs to return to it to retrieve more data.
This approach is better for OLTP workloads because queries typically only access a small number of tuples at a time. Thus, there are fewer function calls to retrieve tuples. The materialization model is not suited for OLAP queries with large intermediate results because the DBMS may have to spill those results to disk between operators.

![11.jpg](assets/1680073298550-0a478dbd-80b9-4dd2-a519-bf4318a35f93.jpeg)

## Vectorization Model

Like the iterator model, each operator in the *vectorization* *model* implements a `Next` function. However, each operator emits a *batch* (i.e. vector) of data instead of a single tuple. The operator’s internal loop implementation is optimized for processing batches of data instead of a single item at a time. The size of the batch can vary based on hardware or query properties. See Figure 3 for an example of the vectorization model.

![12.jpg](assets/1680073299003-10055501-7bd1-40cb-8ee5-14a5731b1383.jpeg)

The vectorization model approach is ideal for OLAP queries that have to scan a large number of tuples because there are fewer invocations of the `Next` function.
The vectorization model allows operators to more easily use **vectorized (SIMD) instructions** to process batches of tuples.

![13.jpg](assets/1680073299509-9a9dea19-46bf-4d92-b57b-07a661bcb9c1.jpeg)

**Figure 3: Vectorization Model Example** – The vectorization model is very similar to the iterator model except at every operator, an output buffer is compared to the desired emission size. If the buffer is larger, then a tuple batch is sent up.

![14.jpg](assets/1680073299487-bbcc90ca-39d0-4b84-9e10-9a7264a86d1c.jpeg)

## Processing Direction

• **Approach #1: Top-to-Bottom**
– Start with the root and “pull” data from children to parents
– Tuples are always passed with function calls

• **Approach #2: Bottom-to-Top**
– Start with leaf nodes and “push” data from children to parents
– Allows for tighter control of caches / registers in operator pipelines

![15.jpg](assets/1680073299507-fde8e559-f89d-4cb8-b209-154a45f25584.jpeg)

# Access Methods

An *access method* is how the DBMS accesses the data stored in a table. In general, there are two approaches to access models; data is either read from a table or from an index with a sequential scan.

![16.jpg](assets/1680073300618-fc8fd7e2-7315-48e9-90c0-fd9deb1700c7.jpeg)

## Sequential Scan

The sequential scan operator iterates over every page in the table and retrieves it from the buffer pool. As the scan iterates over all the tuples on each page, it evaluates the predicate to decide whether or not to emit the tuple to the next operator.
The DBMS maintains an internal cursor that tracks the last page/slot that it examined.

![17.jpg](assets/1680073300844-00e489a7-9f51-445b-b638-cc0a7b6f77dc.jpeg)

A sequential table scan is almost always the least efficient method by which a DBMS may execute a query. There are a number of optimizations available to help make sequential scans faster:
• **Prefetching**: Fetch the next few pages in advance so that the DBMS does not have to block on storage I/O when accessing each page.
• **Buffer Pool Bypass**: The scan operator stores pages that it fetches from disk in its local memory instead of the buffer pool in order to avoid sequential flooding (比如一次scan需要读很多页面，这些页面基本都只用这一次，但是会把很多“热数据页”挤出buffer pool。).
• **Parallelization**: Execute the scan using multiple threads/processes in parallel.
• **Late Materialization**: DSM DBMSs can delay stitching together tuples until the upper parts of the query plan. This allows each operator to pass the minimal amount of information needed to the next operator (e.g. record ID, offset to record in column). This is only useful in column-store systems.
• **Heap Clustering**: Tuples are stored in the heap pages using an order specified by a clustering index.
• **Approximate Queries (Lossy Data Skipping):**  Execute queries on a sampled subset of the entire table to produce approximate results. This is typically done for computing aggregations in a scenario that allow a low error to produce a nearly accurate answer.
• **Zone Map (Lossless Data Skipping):**  Pre-compute aggregations for each tuple attribute **in a page**. The DBMS can then decide whether it needs to access a page by checking its Zone Map first. The Zone Maps for each page are stored in separate pages and there are typically multiple entries in each Zone Map page. Thus, it is possible to reduce the total number of pages examined in a sequential scan. Zone maps are particularly valuable in the cloud database systems where data transfer over a network incurs a bigger cost. See Figure 4 for an example of a Zone Map.

![18.jpg](assets/1680073301495-70b9b450-fb07-4ca4-8fcb-c5a3ec8c7dfc.jpeg)

![19.jpg](assets/1680073302020-37fe695e-9da7-4ff1-90b4-f5f879ec9b3d.jpeg)

![20.jpg](assets/1680073302511-e550800f-8937-4d29-9e08-46a645c14d6d.jpeg)

**Figure 4: Zone Map Example** – The zone map stores pre-computed aggregates for values in a page. In the example above, the select query realizes from the zone map that the max value in the original data is only 400. Then, instead of having to iterate through every tuple in the page, the query can avoid accessing the page at all since none of the values will be greater than 600.

## Index Scan

In an *index scan*, the DBMS picks an index to find the tuples that a query needs.
There are many factors involved in the DBMSs’ index selection process, including:

- What attributes the index contains
- What attributes the query references
- The attribute’s value domains
- Predicate composition（**谓词即函数，返回值是真值**。但是在sql中， 真值并不只是true和false，还包含了第三种情况即：Null。）
- Whether the index has unique or non-unique keys

![21.jpg](assets/1680073302636-d6925914-7695-4fdd-bead-1737a88f9e62.jpeg)

A simple example of an index scan is shown in Figure 5.

![22.jpg](assets/1680073303385-8cd117de-4a92-4f06-a358-38a9a5bdc37e.jpeg)

**Figure 5: Index Scan Example** – Consider a single table with 100 tuples and two indexes: age and department. In the first scenario, it is better to use the department index in the scan because it only has two tuples to match. Choosing the age index would not be much better than a simple sequential scan. In the second scenario, the age index would eliminate more unnecessary scans and is the optimal choice.

![23.jpg](assets/1680073303526-2b7d416b-29f1-42b5-b8e1-b186f6e51dce.jpeg)

More advanced DBMSs support multi-index scans. When using multiple indexes for a query, the DBMS computes sets of record IDs using **each** matching index, **combines** these sets based on the query’s predicates, and retrieves the records and apply any predicates that may remain. The DBMS can use bitmaps, hash tables, or Bloom filters to compute record IDs through set intersection. See Figure 6 for an example that makes use of a multi-index scan.

![24.jpg](assets/1680073304296-3fd35cf2-0f48-4761-9fd8-49d5cb659fb0.jpeg)

![25.jpg](assets/1680073305057-47498033-2c33-4954-9093-27276bcbeb4e.jpeg)

**Figure 6: Multi-Index Scan Example** – Consider the same table in Figure 5. With multi-index scan support, we first compute the sets of record IDs satisfying the predicate for age and dept, respectively, using the corresponding index. We then compute the intersection of the two sets, fetch the corresponding records, and apply the remaining predicate `country=’US’`.

# Modification Queries

Operators that modify the database (`INSERT`, `UPDATE`, `DELETE`) are responsible for checking constraints and updating indexes. For `UPDATE`/`DELETE`, child operators pass Record IDs for target tuples and must keep track of previously seen tuples.

![26.jpg](assets/1680073305431-c72eff3e-20b8-462f-b6c0-66988ebac510.jpeg)

There are two implementation choices on how to handle `INSERT` operators:
•** Choice #1**:# Materialize tuples inside of the operator.
• **Choice #2:**  Operator inserts any tuple passed in from child operators.

![27.jpg](assets/1680073305502-6302921b-7d5e-4c5e-9be8-2ee6863283ee.jpeg)

## Halloween Problem

The Halloween Problem is an anomaly in which an update operation changes the physical location of a tuple, causing a scan operator to visit the tuple multiple times. This can occur on clustered tables or index scans.
This phenomenon was originally discovered by IBM researchers while building **System R** on Halloween day in 1976. The solution to this problem is to keep track of the modified record IDs for each query.

![28.jpg](assets/1680073305801-03c0cc84-1966-4976-a094-ec57a0b3d637.jpeg)

![29.jpg](assets/1680073306472-1fb7e51f-027a-4f59-b6cc-62ee6b63538d.jpeg)

# Expression Evaluation

The DBMS represents a `WHERE` clause as an *expression tree* (see Figure 7 for an example). The nodes in the tree represent different expression types.
Some examples of expression types that can be stored in tree nodes:
• Comparisons (`=`, `<,` `>`, `!=`)
• Conjunction (`AND`), Disjunction (`OR`)
• Arithmetic Operators (`+`, `-`, `*`, `/`, `%`)
• Constant and Parameter Values
• Tuple Attribute References

![30.jpg](assets/1680073307533-da37617b-c746-4bc6-9ba3-dfe647d9fdb5.jpeg)

**Figure 7:**  **Expression Evaluation Example** – A `WHERE` clause and a diagram of its corresponding expression.

![31.jpg](assets/1680073307486-1cfb55e3-16d4-43c0-a341-11b4cb2d2d56.jpeg)

To evaluate an expression tree at runtime, the DBMS maintains a context handle that contains metadata for the execution, such as the current tuple, the parameters, and the table schema. The DBMS then walks the tree to evaluate its operators and produce a result.
Evaluating predicates in this manner is slow because the DBMS must traverse the entire tree and determine the correct action to take for each operator. A better approach is to just **evaluate the expression directly (think JIT compilation)** . Based on a internal cost model, the DBMS would determine whether code generation will be adopted to accelerate a query.

![32.jpg](assets/1680073308074-fe742947-3c82-489e-b147-3035a0ef0e46.jpeg)

![33.jpg](assets/1680073308130-a285d7ec-58f7-420e-b060-77dded89d5a6.jpeg)

![34.jpg](assets/1680073307813-f1e7ac86-348d-4cb3-bb5a-defc2d2dad49.jpeg)

![35.jpg](assets/1680073309749-93e4fca6-ffa9-4aa5-997b-e63d02514aec.jpeg)

![36.jpg](assets/1680073309641-ab5f6224-dde0-44ca-8068-5e6b0e2b852b.jpeg)

![37.jpg](assets/1680073309538-0636cbc5-09e1-4e5a-aac6-5ecd5cde8e25.jpeg)

![38.jpg](assets/1680073309742-9bb54162-4814-4cc9-97a5-78af2ec6e46d.jpeg)

![39.jpg](assets/1680073309948-c905755b-dedf-4e50-b45c-0f6dc6bf5ec0.jpeg)

![40.jpg](assets/1680073311037-c1916331-4158-444b-8cdf-f468d8f6efab.jpeg)

![41.jpg](assets/1680073311188-8345bbea-1f00-4237-b3c8-4c857c776e90.jpeg)

![42.jpg](assets/1680073311443-936f86bb-4a10-43fb-82bc-b0f0f3260902.jpeg)

![43.jpg](assets/1680073311513-8cfc3e38-6fe5-4f16-9d49-afb81070f4e6.jpeg)

![44.jpg](assets/1680073311735-a897ccff-8088-49c8-b00d-753e0e3edff6.jpeg)

![45.jpg](assets/1680073312376-1b94e492-67e2-418c-81c1-f1a12c93f36e.jpeg)
