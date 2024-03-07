# 13 - Parallel Query Execution

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103633406-a2cae6f3-f3ab-4408-8d5e-71bcb9cf9470.jpeg#averageHue=%231c2635&clientId=uc27adca1-91a6-4&from=ui&id=u0a5f94e2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=598715&status=done&style=none&taskId=u7cf548d7-6405-4ee6-ab60-cfc6402ec03&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103632676-321d71d6-1d3a-43fb-acdb-0b3184730961.jpeg#averageHue=%23eeeeee&clientId=uc27adca1-91a6-4&from=ui&id=u9337320b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=261943&status=done&style=none&taskId=uc5249145-2945-4422-bd1a-cb13107239b&title=)

# Background

Previous discussions of query executions assumed that the queries executed with a single worker (i.e thread). However, in practice, queries are often executed in parallel with multiple workers.

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103633051-ba7ee4e1-b5af-46e5-883d-27281084cc93.jpeg#averageHue=%23e3e3e3&clientId=uc27adca1-91a6-4&from=ui&id=u92fc4fba&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=398019&status=done&style=none&taskId=u1724a829-46a3-496d-8a1d-7ad806e3a24&title=)

Parallel execution provides a number of key benefits for DBMSs:
• Increased performance in throughput (more queries per second) and latency (less time per query).
• Increased responsiveness and availability from the perspective of external clients of the DBMS.
• Potentially lower *total cost of ownership* (TCO). This cost includes both the hardware procurement and software license, as well as the labor overhead of deploying the DBMS and the energy needed to run the machines.
There are two types of parallelism that DBMSs support: inter-query parallelism and intra-query parallelism.

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103632982-e802ec3e-82b0-4f8c-80f8-baaea06760ee.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=uf83518a8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=328121&status=done&style=none&taskId=uec7e8851-d11a-4067-9b75-fe77136e7d3&title=)

# Parallel vs Distributed Databases

In both parallel and distributed systems, the database is spread out across multiple “resources” to improve parallelism. These resources may be computational (e.g., CPU cores, CPU sockets, GPUs, additional machines) or storage (e.g., disks, memory).
It is important to distinguish between parallel and distributed systems.
• **Parallel DBMS** In a parallel DBMS, resources, or nodes, are physically close to each other. These nodes communicate with high-speed interconnect. It is assumed that communication between resources is not only fast, but also cheap and reliable.
• **Distributed DBMS** In a distributed DBMS, resources may be far away from each other; this might mean the database spans racks or data centers in different parts of the world. As a result, resources communicate using a slower interconnect over a public network. Communication costs between nodes are higher and failures cannot be ignored.

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103632954-1845bf29-5301-420c-b11d-fca9a366a368.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=u578fae1b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=336145&status=done&style=none&taskId=uea72c918-097f-493b-83b7-830aa323860&title=)

Even though a database may be physically divided over multiple resources, it still appears as a single logical database instance to the application. Thus, a SQL query executed against a single-node DBMS should generate the same result on a parallel or distributed DBMS.

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103634686-235c3757-74cc-40b4-91b0-06f8669b7725.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=u46338b10&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=335434&status=done&style=none&taskId=u4a7c7cd9-ad36-4f7f-940c-12596b6a06b&title=)

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103634502-e6da84a2-5fd4-4582-880b-35043bfa7c67.jpeg#averageHue=%23f0f0f0&clientId=uc27adca1-91a6-4&from=ui&id=u393de2a6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=174347&status=done&style=none&taskId=u95d1dae4-2597-43cf-9ae0-8f6215c1276&title=)

# Process Models

A DBMS *process model* defines how the system supports concurrent requests from a multi-user application/environment. The DBMS is comprised of more or more *workers* that are responsible for executing tasks on behalf of the client and returning the results. An application may send a large request or multiple requests at the same time that must be divided across different workers.
There are two major process models that a DBMS may adopt: process per worker and thread per worker. A third common database usage pattern takes an embedded approach.

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103634883-bacdaa18-8ef1-45b7-a6a8-7d657ec5be26.jpeg#averageHue=%23ececec&clientId=uc27adca1-91a6-4&from=ui&id=ub3ba8fbd&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=309155&status=done&style=none&taskId=u7dd7edfa-4ce5-42d0-a529-d672933b8fe&title=)

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103635168-208b5d3b-7098-4ad4-b139-0915254fbd41.jpeg#averageHue=%23ededed&clientId=uc27adca1-91a6-4&from=ui&id=u436c4406&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=224426&status=done&style=none&taskId=ub0b406c6-6435-4340-8e01-3013790faf4&title=)

## Process per Worker

The most basic approach is *process per worker*. Here, each worker is a separate OS process, and thus relies on OS scheduler. An application sends a request and opens a connection to the databases system. Some **dispatcher** receives the request selects one of its worker processes to manage the connection. The application now communicates directly with the worker who is responsible for executing the request that the query wants. This sequence of events is shown in Figure 1.
Relying on the operating system for scheduling effectively reduces the DBMS’s control over execution. Further, this model depends on shared memory to maintain global data structures or relies on message passing, which has higher overhead.
An advantage of the process per worker approach is that a process crash doesn’t disrupt the whole system because each worker runs in the context of its own OS process.
This process model raises the issue of multiple workers on separate processes making numerous copies of the same page. A solution to maximize memory usage is to use shared-memory for global data structures so that they can be shared by workers running in different processes.
Examples of systems that utilize the process-per-worker process model include IBM DB2, Postgres, and Oracle. When these DBMSs were developed, pthreads had not yet become the standard threading model. The semantics of threading varied from OS to OS while `fork()` was better defined.

![Figure 1: Process per Worker Model](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103635536-4d112fff-3315-4c46-894b-ff4e0723e869.jpeg#averageHue=%23e5e4e4&clientId=uc27adca1-91a6-4&from=ui&id=uecc4a102&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=true&size=380171&status=done&style=none&taskId=uaf44595c-34f2-4d7b-bab7-876d33b8153&title=Figure%201%3A%20Process%20per%20Worker%20Model)

## Thread per Worker

The most common model nowadays is *thread per worker*. Instead of having different processes doing different tasks, each database system has only one process with multiple worker threads. In this environment, the DBMS has full control over the tasks and threads, it can manage it own scheduling. The multi-threaded model may or may not use a dispatcher thread. A diagram of the thread per worker model is shown in Figure 2.
Using multi-threaded architecture provides certain advantages. For one, there is less overhead per context switch. Additionally, a shared model does not have to be maintained. However, the thread per worker model does not necessarily imply that the DBMS supports intra-query parallelism.
Almost every DBMS created in the last 20 years uses this approach, including Microsoft SQL Server and MySQL. IBM DB2 and Oracle have updated their models to provide support for this apporach, as well. Postgres and Postgres-dervied databases largely still use the process-based approach.

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103636752-097be003-98a6-46dc-bf98-3090986602ea.jpeg#averageHue=%23e4e3e3&clientId=uc27adca1-91a6-4&from=ui&id=ud9b0ff5f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=444976&status=done&style=none&taskId=u80ec9ba2-4ba8-4ff2-8f80-7d8354f1d04&title=)

## Scheduling

In conclusion, for each query plan, the DBMS has to decide where, when, and how to execute. Relevant questions include:
• How many tasks should it use?
• How many CPU cores should it use?
• What CPU cores should the tasks execute on?
• Where should a task store its output?
When making decisions regarding query plans, the DBMS **always** knows more than the OS and should be prioritized as such.

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103636675-b0c56aae-4c32-4d41-9fff-99cdbc91dd70.jpeg#averageHue=%23ededed&clientId=uc27adca1-91a6-4&from=ui&id=u9beed923&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=295028&status=done&style=none&taskId=ubd00d175-b105-4b97-961e-ffd4b276398&title=)

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103636981-2fd998c3-9579-49ca-9ec2-4ab1eb8fc877.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=u6390ff92&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=345612&status=done&style=none&taskId=u8cd311bb-0663-49d4-8ff7-a637f113c7c&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103637684-5f268a96-ef71-47ef-877a-fd6437cea9db.jpeg#averageHue=%23efefee&clientId=uc27adca1-91a6-4&from=ui&id=u7494b20e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=617418&status=done&style=none&taskId=u239c059d-6906-4d39-94ac-0fe5fc93789&title=)

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103637418-d7c9fec9-5f6e-44b2-9ca3-296916b2d6cf.jpeg#averageHue=%23e8e7e7&clientId=uc27adca1-91a6-4&from=ui&id=u8a080b7b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=322725&status=done&style=none&taskId=u09c8d424-0b4b-4efc-9024-3fc73b42ac6&title=)

## Embedded DBMS

A very different usage pattern for databases invovles running the system in the same address space of the application, as opposed to a client-server model where the database stands independent of the application. In this scenario, the application will set up the threads and tasks to run on the database system. The application itself will largely be responsible for scheduling. A diagram of an embedded DBMS’s scheduling behaviors is shown in Figure 3.
DuckDB, SQLite, and RocksDB are the most famous embedded DBMSs.

![Figure 3: Embedded DBMS Scheduling](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103638675-74f91549-e172-4cd2-a7c6-6df7d33ce092.jpeg#averageHue=%23e7e7e7&clientId=uc27adca1-91a6-4&from=ui&id=ub0f304a8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=true&size=378345&status=done&style=none&taskId=u6342dc1a-0244-47eb-ae35-49a78c87bde&title=Figure%203%3A%20Embedded%20DBMS%20Scheduling)

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103639170-eeb97c95-7c70-4ef4-9551-731bf21e8b0a.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=u594bcf39&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=361230&status=done&style=none&taskId=u0a52b693-f669-41ae-b885-800d90f5ef4&title=)

# Inter-Query Parallelism

In *inter-query parallelism*, the DBMS executes **different** queries are concurrently. Because multiple workers are running requests simultaneously, overall performance is improved. This increases throughput and reduces latency.
If the queries are read-only, then little coordination is required between queries. However, if multiple queries are updating the database concurrently, more complicated conflicts arise. These issues are discussed further in lecture 15.

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103639144-c67aabda-b2ce-48ef-8b45-627ad56cae01.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=ucb5d7d2e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=305178&status=done&style=none&taskId=uf02c04aa-dd12-4d15-aeae-8e1e9d745ed&title=)

# Intra-Query parallelism

In *intra-query parallelism*, the DBMS executes the operations of a **single** query in parallel. This decreases latency for long-running queries.
The organization of intra-query parallelism can be thought of in terms of a producer/consumer paradigm. Each operator is a producer of data as well as a consumer of data from some operator running below it.

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103639635-54d409e8-476e-4210-963c-0436bc9dd0d0.jpeg#averageHue=%23eaeaea&clientId=uc27adca1-91a6-4&from=ui&id=u6ecc2cc0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=396474&status=done&style=none&taskId=u9187c479-8255-48a3-886a-b486f27bec1&title=)

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103640111-fcbb95c3-63b1-4fcc-929d-1c6a44d97381.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=u4d090a08&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=337745&status=done&style=none&taskId=uf9af129a-1630-490b-973e-da083328c2b&title=)

Parallel algorithms exist for every relational operator. The DBMS can either have multiple threads access centralized data structures or use partitioning to divide work up.

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103640730-9ea7fec4-126e-4f8b-b558-217bfc8d25e0.jpeg#averageHue=%23eae9e9&clientId=uc27adca1-91a6-4&from=ui&id=u6fecd89c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=359947&status=done&style=none&taskId=ue1438155-7759-43a0-9b6c-1a96a893106&title=)

Within intra-query parallelism, there are three types of parallelism: intra-operator, inter-operator, and bushy. These approaches are not mutually exclusive. It is the DBMS’ responsibility to combine these techniques in a way that optimizes performance on a given workload.

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103641017-2e4918cc-e82c-410f-9820-f88418da28a2.jpeg#averageHue=%23ededed&clientId=uc27adca1-91a6-4&from=ui&id=u75b3d3db&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=222111&status=done&style=none&taskId=u93198656-820d-4630-83d8-df129967950&title=)

## Intra-Operator Parallelism (Horizontal)

In *intra-operator parallelism*, the query plan’s operators are decomposed into independent fragments that perform the same function on different (disjoint) subsets of data.
The DBMS inserts an exchange operator into the query plan to coalesce results from child operators. The exchange operator prevents the DBMS from executing operators above it in the plan until it receives all of the data from the children. An example of this is shown in Figure 4.

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103641316-a7108a30-9cea-4c8b-8d50-30afcd193afd.jpeg#averageHue=%23eaeaea&clientId=uc27adca1-91a6-4&from=ui&id=ue34e33c7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=347861&status=done&style=none&taskId=u67191ce7-c928-4764-8133-c33ee9d3c6b&title=)

![Figure 4: Intra-Operator Parallelism – The query plan for this SELECT is a sequential scan on A that is fed into a filter operator. To run this in parallel, the query plan is partitioned into disjoint fragments. A given plan fragment is operated on a by a distinct worker. The exchange operator calls Next concurrently on all fragments which then retrieve data from their respective pages.](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103641488-e0c6e1a9-0154-4408-b3bc-532d6a350d99.jpeg#averageHue=%23dcdbdb&clientId=uc27adca1-91a6-4&from=ui&id=u81e55dc8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=true&size=329741&status=done&style=none&taskId=u05c89fc5-5553-48ab-a96f-4e9dd3b00f7&title=Figure%204%3A%20Intra-Operator%20Parallelism%20%E2%80%93%20The%20query%20plan%20for%20this%20SELECT%20is%20a%20sequential%20scan%20on%20A%20that%20is%20fed%20into%20a%20filter%20operator.%20To%20run%20this%20in%20parallel%2C%20the%20query%20plan%20is%20partitioned%20into%20disjoint%20fragments.%20A%20given%20plan%20fragment%20is%20operated%20on%20a%20by%20a%20distinct%20worker.%20The%20exchange%20operator%20calls%20Next%20concurrently%20on%20all%20fragments%20which%20then%20retrieve%20data%20from%20their%20respective%20pages.)

In general, there are three types of exchange operators:
• **Gather:**  Combine the results from multiple workers into a single output stream. This is the most common type used in parallel DBMSs.
• **Distribute: **Split a single input stream into multiple output streams.
• **Repartition: **Reorganize multiple input streams across multiple output streams. This allows the DBMS take inputs that are partitioned one way and then redistribute them in another way.

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103642094-77aeb80a-c4c0-4d11-9989-3be60976ff06.jpeg#averageHue=%23e4e3e3&clientId=uc27adca1-91a6-4&from=ui&id=u7c5ee4c9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=409946&status=done&style=none&taskId=ub64e5cf8-c75f-4f25-94c1-4557a4c984e&title=)

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103642806-2317590e-eaec-4e23-9208-9139ee6abd0f.jpeg#averageHue=%23d7d6d6&clientId=uc27adca1-91a6-4&from=ui&id=uc8d3a614&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=396967&status=done&style=none&taskId=ua2b759ed-a4a3-4fda-9586-a5aec9c1dbe&title=)

## Inter-Operator Parallelism (Vertical)

In *inter-operator parallelism*, the DBMS overlaps operators in order to pipeline data from one stage to the next without materialization. This is sometimes called pipelined parallelism. See example in Figure 5.
This approach is widely used *in stream processing systems*, which are systems that continually execute a query over a stream of input tuples.

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103643209-bbf93ff5-e58a-4995-aad8-527d247314df.jpeg#averageHue=%23eaeaea&clientId=uc27adca1-91a6-4&from=ui&id=uff616cf8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=386350&status=done&style=none&taskId=uffaa916e-9add-4738-90b3-a4c70458c3e&title=)

![Figure 5: Inter-operator Parallelism – In the JOIN statement to the left, a single worker performs the join and then emits the result to another worker that performs the projection and then emits the result again.](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103643409-80d8b678-af41-46d4-9d8e-c8a929bc0dd5.jpeg#averageHue=%23dedede&clientId=uc27adca1-91a6-4&from=ui&id=u418db301&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=true&size=331345&status=done&style=none&taskId=u618f26ad-a402-49ad-bf49-9f3ddaebfa2&title=Figure%205%3A%20Inter-operator%20Parallelism%20%E2%80%93%20In%20the%20JOIN%20statement%20to%20the%20left%2C%20a%20single%20worker%20performs%20the%20join%20and%20then%20emits%20the%20result%20to%20another%20worker%20that%20performs%20the%20projection%20and%20then%20emits%20the%20result%20again.)

## Bushy Parallelism

*Bushy parallelism* is a hybrid of intra-operator and inter-operator parallelism where workers execute multiple operators from different segments of the query plan at the same time.
The DBMS still uses exchange operators to combine intermediate results from these segments. An example is shown in Figure 6.

![Figure 6: Bushy Parallelism – To perform a 4-way JOIN on three tables, the query plan is divided into four fragments as shown. Different portions of the query plan run at the same time, in a manner similar to inter-operator parallelism.](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103643632-0f082526-3a15-49d1-b129-448e7dead0ef.jpeg#averageHue=%23e1e1e1&clientId=uc27adca1-91a6-4&from=ui&id=u7f490b95&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=true&size=440359&status=done&style=none&taskId=ub12cf4de-ec86-4a7c-b939-2770c8e7e0e&title=Figure%206%3A%20Bushy%20Parallelism%20%E2%80%93%20To%20perform%20a%204-way%20JOIN%20on%20three%20tables%2C%20the%20query%20plan%20is%20divided%20into%20four%20fragments%20as%20shown.%20Different%20portions%20of%20the%20query%20plan%20run%20at%20the%20same%20time%2C%20in%20a%20manner%20similar%20to%20inter-operator%20parallelism.)

# I/O Parallelism

Using additional processes/threads to execute queries in parallel will not improve performance if the disk is always the main bottleneck. Therefore, it is important to be able to split a database across multiple storage devices.
To get around this, DBMSs use I/O parallelism to *split installation across multiple devices.*  Two approaches to I/O parallelism are multi-disk parallelism and database partitioning.

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103643771-55a01077-e358-4e97-acd4-8e752645ee1e.jpeg#averageHue=%23ececec&clientId=uc27adca1-91a6-4&from=ui&id=u6c9316a2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=297708&status=done&style=none&taskId=ud5244163-a910-49d9-87d9-9c923a9b88f&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103644573-43cc7fa6-a347-45de-acb6-2f1c0f8619f3.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=u74c6d683&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=351683&status=done&style=none&taskId=u089ada44-8646-4a38-b283-af9e2519b7d&title=)

## Multi-Disk Parallelism

In *multi-disk parallelism*, the OS/hardware is configured to store the DBMS’s files across multiple storage devices. This can be done through storage appliances or [RAID configuration](http://www.imooc.com/article/264962). All of the storage setup is transparent to the DBMS so workers cannot operate on different devices because the DBMS is unaware of the underlying parallelism.

### RAID 是什么？

RAID （ Redundant Array of Independent Disks ）即独立磁盘冗余阵列，简称为「磁盘阵列」，其实就是用多个独立的磁盘组成在一起形成一个大的磁盘系统，从而实现比单块磁盘更好的存储性能和更高的可靠性。

### RAID0

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22382307/1680156600305-a8149a1d-f64f-4996-ab9b-03aab319d74a.png#averageHue=%23b2b3b0&clientId=uc27adca1-91a6-4&from=paste&height=477&id=QKcDg&originHeight=424&originWidth=500&originalType=binary&ratio=2&rotation=0&showTitle=false&size=135524&status=done&style=none&taskId=u2db5bdf4-178f-446a-aa6b-07264443f85&title=&width=563)

RAID0 是一种非常简单的的方式，它将多块磁盘组合在一起形成一个大容量的存储。当我们要写数据的时候，会将数据分为$N$份，以独立的方式实现$N$块磁盘的读写，那么这N份数据会同时并发的写到磁盘中，因此执行性能非常的高。
RAID0 的读写性能理论上是单块磁盘的$N$倍（仅限理论，因为实际中磁盘的寻址时间也是性能占用的大头）
但RAID0的问题是，它并不提供数据校验或冗余备份，因此一旦某块磁盘损坏了，数据就直接丢失，无法恢复了。因此RAID0就不可能用于高要求的业务中，但可以用在对可靠性要求不高，对读写性能要求高的场景中。

### RAID1

RAID1 是磁盘阵列中单位成本最高的一种方式。因为它的原理是在往磁盘写数据的时候，将同一份数据无差别的写两份到磁盘，分别写到工作磁盘和镜像磁盘，那么它的实际空间使用率只有50%了，两块磁盘当做一块用，这是一种比较昂贵的方案。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22382307/1680156622658-ce53c525-0c8e-4e58-bd89-9ad552297579.png#averageHue=%23b9bbb7&clientId=uc27adca1-91a6-4&from=paste&height=536&id=uc399bcdd&originHeight=467&originWidth=500&originalType=binary&ratio=2&rotation=0&showTitle=false&size=161183&status=done&style=none&taskId=ude947dd4-547e-4022-8f3d-5e6963853ec&title=&width=574)

RAID1其实与RAID0效果刚好相反。RAID1 这种写双份的做法，就给数据做了一个冗余备份。这样的话，任何一块磁盘损坏了，都可以再基于另外一块磁盘去恢复数据，数据的可靠性非常强，但性能就没那么好了。

### RAID5

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22382307/1680156658168-5c9fd839-14e3-4e58-b0a0-e702877437c3.png#averageHue=%23dadbd8&clientId=uc27adca1-91a6-4&from=paste&height=371&id=u5a0c5867&originHeight=302&originWidth=500&originalType=binary&ratio=2&rotation=0&showTitle=false&size=93910&status=done&style=none&taskId=ue81dbf17-d3e3-43e3-8563-a097c4ec664&title=&width=615)

因为 RAID5 是一种将 存储性能、数据安全、存储成本 兼顾的一种方案。
在了解RAID5之前，我们可以先简单看一下RAID3，虽然RAID3用的很少，但弄清楚了RAID3就很容易明白RAID5的思路。
RAID3的方式是：将数据按照RAID0的形式，分成多份同时写入多块磁盘，但是还会另外再留出一块磁盘用于写「奇偶校验码」。例如总共有$N$块磁盘，那么就会让其中额度$N-1$块用来并发的写数据，第$N$块磁盘用记录校验码数据。一旦某一块磁盘坏掉了，就可以利用其它的$N-1$块磁盘去恢复数据。
但是由于第$N$块磁盘是校验码磁盘，因此有任何数据的写入都会要去更新这块磁盘，导致这块磁盘的读写是最频繁的，也就非常的容易损坏。
RAID5的方式可以说是对RAID3进行了改进。
RAID5模式中，不再需要用单独的磁盘写校验码了。它把校验码信息分布到各个磁盘上。例如，总共有$N$块磁盘，那么会将要写入的数据分成$N$份，并发的写入到$N$块磁盘中，同时还将数据的校验码信息也写入到这$N$块磁盘中（**数据与对应的校验码信息必须得分开存储在不同的磁盘上**）。一旦某一块磁盘损坏了，就可以用剩下的数据和对应的奇偶校验码信息去恢复损坏的数据。
RAID5校验位算法原理：$P = D1 \oplus D2 \oplus D3 … \oplus Dn$ （D1,D2,D3 … Dn为数据块，P为校验，xor为异或运算）
RAID5的方式，最少需要三块磁盘来组建磁盘阵列，允许最多同时坏一块磁盘。如果有两块磁盘同时损坏了，那数据就无法恢复了。

### RAID6

为了进一步提高存储的高可用，聪明的人们又提出了RAID6方案，可以在有两块磁盘同时损坏的情况下，也能保障数据可恢复。
为什么RAID6这么牛呢，因为RAID6在RAID5的基础上再次改进，引入了**双重校验**的概念。
RAID6除了每块磁盘上都有同级数据$XOR$校验区以外，还有针对每个数据块的$XOR$校验区，这样的话，相当于每个数据块有两个校验保护措施，因此数据的冗余性更高了。
但是RAID6的这种设计也带来了很高的复杂度，虽然数据冗余性好，读取的效率也比较高，但是写数据的性能就很差。因此RAID6在实际环境中应用的比较少。

### RAID10

RAID10其实就是RAID1与RAID0的一个合体。我们看图就明白了：

![](https://cdn.nlark.com/yuque/0/2023/webp/22382307/1680157060351-af156b07-8dee-4075-a4d3-735a73e0d7a3.webp#averageHue=%23fbfcf9&clientId=uc27adca1-91a6-4&from=paste&height=400&id=ueb1f7516&originHeight=307&originWidth=500&originalType=url&ratio=2&rotation=0&showTitle=false&status=done&style=none&taskId=ub1d91daa-54c5-4422-8597-ed8dd248834&title=&width=652)

RAID10兼备了RAID1和RAID0的有优点。首先基于RAID1模式将磁盘分为2份，当要写入数据的时候，将所有的数据在两份磁盘上同时写入，相当于写了双份数据，起到了数据保障的作用。且在每一份磁盘上又会基于RAID0技术讲数据分为N份并发的读写，这样也保障了数据的效率。
但也可以看出RAID10模式是有一半的磁盘空间用于存储冗余数据的，浪费的很严重，因此用的也不是很多。
**整体对比一下** RAID0、RAID1、RAID5、RAID6、RAID10 的几个特征：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/22382307/1680157163416-719b3247-e84a-418d-af62-e3bb9ce8d6f2.png#averageHue=%23f2f2f2&clientId=uc27adca1-91a6-4&from=paste&height=265&id=u36e75187&originHeight=363&originWidth=1007&originalType=binary&ratio=2&rotation=0&showTitle=false&size=108966&status=done&style=none&taskId=u20c067e7-211a-4de9-afd3-0fc1a1a44f2&title=&width=736.5)

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103645450-51603955-5137-4639-88fc-9aa7553f2880.jpeg#averageHue=%23e6e6e6&clientId=uc27adca1-91a6-4&from=ui&id=u716516da&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=354837&status=done&style=none&taskId=uc4ebdc51-20b1-4261-acc6-9901416dabe&title=)

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103645594-7f4ffb5b-8109-407f-a7c0-aa328d9dc559.jpeg#averageHue=%23e9e8e8&clientId=uc27adca1-91a6-4&from=ui&id=ue8c886c8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=355635&status=done&style=none&taskId=uf8ce9f2e-386c-4197-8704-2dd96727370&title=)

## Database Partitioning

In *database partitioning*, the database is split up into disjoint subsets that can be assigned to discrete disks. Some DBMSs allow for specification of the disk location of each individual database. This is easy to do at the file-system level if the DBMS stores each database in a separate directory. The log file of changes made is usually shared.

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103645798-f120235b-7b55-4ca8-aa62-f17f9bc70e67.jpeg#averageHue=%23ebebeb&clientId=uc27adca1-91a6-4&from=ui&id=u696412c2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=353070&status=done&style=none&taskId=u0b6d85ca-e553-4e9c-a9dc-82b5604e346&title=)

The idea of _logical partitioning _is to split single logical table into disjoint physical segments that are stored/managed separately. Such partitioning is ideally transparent to the application. That is, the application should be able to access logical tables without caring how things are stored.
The two approaches to partitioning are vertical and horizontal partitioning.
In *vertical partitioning*, a table’s attributes are stored in a separate location (like a column store). The tuple information must be stored in order to reconstruct the original record.
In *horizontal partitioning,*  the tuples of a table are divided into disjoint segments based on some partitioning keys. There are different ways to decide how to partition (e.g., hash, range, or predicate partitioning). The efficacy of each approach depends on the queries.
We will cover these approaches later in the semester when discussing distributed databases.

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103645864-3aaadec0-7aa6-4099-ba6c-73676b03fa72.jpeg#averageHue=%23eaeaea&clientId=uc27adca1-91a6-4&from=ui&id=u06c562a3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=370288&status=done&style=none&taskId=u3b91f29b-38ba-470d-80de-a20f30bd52b&title=)

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103646279-1d978157-0e0d-42fb-a2d8-a1b093a0b7c6.jpeg#averageHue=%23eeeeee&clientId=uc27adca1-91a6-4&from=ui&id=u99236910&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=257324&status=done&style=none&taskId=ua9942b85-398c-4fbc-bb02-1134783f56b&title=)

![37.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1680103647579-392f67c3-e1cf-4e98-b8b0-47e21e650fc0.jpeg#averageHue=%23eeecec&clientId=uc27adca1-91a6-4&from=ui&id=u9221be4e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=349181&status=done&style=none&taskId=u00e334d7-53ca-4fde-a633-537b5215afb&title=)
