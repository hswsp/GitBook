# 09 - Concurrent Indexes

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371877798-b949adc3-ec39-42dd-8a94-3b8f0622d1b7.jpeg#averageHue=%231d2635\&clientId=u23110e91-7f27-4\&from=ui\&id=ud910beab\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=600047\&status=done\&style=none\&taskId=u83e6f7e7-c688-4de3-9e89-ee0e73d6929\&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371877413-60ac4f29-ec8a-490f-b136-b180e970ca13.jpeg#averageHue=%23efefef\&clientId=u23110e91-7f27-4\&from=ui\&id=ua490e647\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=197379\&status=done\&style=none\&taskId=u611ddee1-4fec-4079-9110-134493fbc04\&title=)

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371877672-14a542ba-33bb-4d18-8231-fcfc8759e377.jpeg#averageHue=%23ecebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u8462100c\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=361189\&status=done\&style=none\&taskId=u92370d94-7e2f-46e1-861e-67e6bce6d47\&title=)

## Index Concurrency Control

So far, we assumed that the data structures we have discussed are single-threaded. However, most DBMSs needs to allow multiple threads to safely access data structures to take advantage of additional CPU cores and hide disk I/O stalls. A _concurrency control_ protocol is the method that the DBMS uses to ensure “correct” results for concurrent operations on a shared object. A protocol’s correctness criteria can vary:

* **Logical Correctness**: This means that the thread is able to read values that it should expects to read, e.g. a thread should read back the value it had written previously.
* **Physical Correctness**: This means that the internal representation of the object is sound, e.g. there are not pointers in the data structure that will cause a thread to read invalid memory locations.

For the purposes of this lecture, we only care about enforcing **physical correctness**. We will revisit logical correctness in later lectures.

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371877707-0b72b991-4b41-42a4-980f-3ebd247527b8.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u08484724\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=375077\&status=done\&style=none\&taskId=ud49d089f-0764-47f0-9425-64c04bc8ca5\&title=)

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371877479-db70cf73-8564-4c20-814b-47c9d72d7f04.jpeg#averageHue=%23efefef\&clientId=u23110e91-7f27-4\&from=ui\&id=u9fe78505\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=186524\&status=done\&style=none\&taskId=u8c7402f2-6c50-4568-93d5-c3640446c42\&title=)

## Locks vs. Latches

There is an important distinction between locks and latches when discussing how the DBMS protects its internal elements.

### Locks

A lock is a higher-level, logical primitive that protects the contents of a database (e.g., tuples, tables, databases) from other transactions. Transactions will hold a lock for its entire duration. Database systems can expose to the user the locks that are being held as queries are run. **Locks need to be able to rollback changes.**

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371880280-3207d9cd-ddb7-4dad-9898-fb038d50bd1a.jpeg#averageHue=%23ededed\&clientId=u23110e91-7f27-4\&from=ui\&id=u8e8f9e1d\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=296499\&status=done\&style=none\&taskId=u09d825ee-804e-4250-a40c-7830b049f5d\&title=)

### Latches

Latches are the low-level protection primitives used for critical sections the DBMS’s internal data structures (e.g., data structure, regions of memory) from other threads. Latches are held for only the duration of the operation being made. Latches do not need to be able to rollback changes. There are two modes for latches:

* **READ:** Multiple threads are allowed to read the same item at the same time. A thread can acquire the read latch even if another thread has acquired it as well.
* \*\*WRITE: \*\*Only one thread is allowed to access the item. A thread cannot acquire a write latch if another thread holds the latch in any mode. A thread holding a write latch also prevents other threads from acquiring a read latch.

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371880277-cb5b2216-ae03-4f28-a40e-3cbd947e8dee.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u2641d920\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=404247\&status=done\&style=none\&taskId=u42f35975-02dc-48da-b2dc-4b8fae07860\&title=)

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371880254-d38b06dc-334d-4919-8b1e-6bbd6db517b4.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u3f18aee9\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=334623\&status=done\&style=none\&taskId=u533a2036-12a4-4399-ac05-738157f7511\&title=)

## Latch Implementations

The underlying primitive that used to implement a latch is through an atomic compare-and-swap (**CAS**) instruction that modern CPUs provide. With this, a thread can check the contents of a memory location to see whether it has a certain value. If it does, then the CPU will swap the old value with a new one. Otherwise the memory location remains unmodiﬁed. There are several approaches to implementing a latch in a DBMS. Each approach has different trade-offs in terms of engineering complexity and runtime performance. These test-and-set steps are performed atomically (i.e., no other thread can update the value in between the test and set steps.

### Blocking OS Mutex

One possible implementation of latches is the OS built-in mutex infrastructure. Linux provides the `**futex**` (fast user-space mutex), which is comprised of (1) a spin latch in user-space and (2) an OS-level mutex. If the DBMS can acquire the user-space latch, then the latch is set. _It appears as a single latch to the DBMS even though it contains two internal latches._ If the DBMS fails to acquire the user-space latch, then it goes down into the kernel and tries to acquire a more expensive mutex. If the DBMS fails to acquire this second mutex, then the thread notiﬁes the OS that it is blocked on the mutex and then it is descheduled. **OS mutex is generally a bad idea** inside of DBMSs as it is managed by OS and has large overhead.

* **Example**: `std::mutex`
* **Advantages**: Simple to use and requires no additional coding in DBMS.
* **Disadvantages**: Expensive and non-scalable (about 25 ns per lock/unlock invocation) because of OS scheduling.

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371880254-771b7c98-3125-4c46-ab13-a955b5fb4d44.jpeg#averageHue=%23e6e5e5\&clientId=u23110e91-7f27-4\&from=ui\&id=u8df61eae\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=339064\&status=done\&style=none\&taskId=u221a6108-acae-4be1-a735-86eff24e922\&title=)

### Reader-Writer Latches

Mutexes and Spin Latches do not differentiate between reads/writes (i.e., they do not support different modes). The DBMS needs a way to allow for concurrent reads, so if the application has heavy reads it will have better performance because readers can share resources instead of waiting. A Reader-Writer Latch allows a latch to be held in either read or write mode. It keeps track of how many threads hold the latch and are waiting to acquire the latch in each mode. Reader-writer latches use one of the previous two latch implementations as primitives and have additional logic to handle **reader-writer queues**, which are queues requests for the latch in each mode. Different DBMSs can have different policies for how it handles the queues.

* **Example**: `std::shared mutex`
* **Advantages**: Allows for concurrent readers.
* **Disadvantages**: The DBMS has to manage read/write queues to avoid starvation. Larger storage overhead than spin Latches due to additional meta-data.

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371880231-248bb10d-7538-4893-8aa4-b322e8abfba3.jpeg#averageHue=%23e9e9e9\&clientId=u23110e91-7f27-4\&from=ui\&id=ua3084fb2\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=309248\&status=done\&style=none\&taskId=u7528161c-f9cd-4abb-9906-730da2e5fbe\&title=)

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371881757-5aad7adb-e2d3-440a-bfd6-1ca7113c5637.jpeg#averageHue=%23e9e9e9\&clientId=u23110e91-7f27-4\&from=ui\&id=u46c5771b\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=311367\&status=done\&style=none\&taskId=u48fcca33-9241-43e9-9302-adac1af7f4a\&title=)

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371882219-f0f9f60c-34e0-4c55-9217-dd3bf71026cb.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u2c6e1f49\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=320642\&status=done\&style=none\&taskId=u5b114df4-ac84-47c9-a03e-51ed4ee34b5\&title=)

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371882334-aa709911-8189-4817-b228-acede5615a99.jpeg#averageHue=%23e7e7e7\&clientId=u23110e91-7f27-4\&from=ui\&id=uc2989b0c\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=330042\&status=done\&style=none\&taskId=ubd6fe34e-7c52-45b6-9570-ab785ffeae6\&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371882365-e1faca4f-d916-488d-a0c1-48d19ac55563.jpeg#averageHue=%23e6e6e6\&clientId=u23110e91-7f27-4\&from=ui\&id=ud0992b6f\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=339540\&status=done\&style=none\&taskId=ub3369375-aae1-4646-a675-19ad667550b\&title=)

### Test-and-Set Spin Latch (TAS)

Spin latches are a more efﬁcient alternative to an OS mutex as it is controlled by the DBMSs. A spin latch is essentially a location in memory that threads try to update (e.g., setting a boolean value to true). A thread performs CAS to attempt to update the memory location. The DBMS can control what happens if it fails to get the latch. It can choose to try again (for example, using a while loop) or allow the OS to deschedule it. Thus, this method gives the DBMS more control than the OS mutex, where failing to acquire a latch gives control to the OS.

* **Example**: `std::atomic<T>`
* **Advantages**: Latch/unlatch operations are efﬁcient (single instruction to lock/unlock).
* **Disadvantages**: Not scalable nor cache-friendly because with multiple threads, the CAS instructions will be executed multiple times in different threads. These wasted instructions will pile up in high contention environments; the threads look busy to the OS even though they are not doing useful work. This leads to cache coherence problems because threads are polling cache lines on other CPUs.

## Hash Table Latching

It is easy to support concurrent access in a static hash table due to the limited ways threads access the data structure. For example, all threads move in the same direction when moving from slot to the next (i.e., top-down). Threads also only access a single page/slot at a time. Thus, deadlocks are not possible in this situation because no two threads could be competing for latches held by the other. When we need to resize the table, we can just take a global latch on the entire table to perform the operation. Latching in a dynamic hashing scheme (e.g., extendible) is a more complicated scheme because there is more shared state to update, but the general approach is the same.

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371882381-dcec94be-86d2-4b41-9de7-4d6cb21269a9.jpeg#averageHue=%23ececec\&clientId=u23110e91-7f27-4\&from=ui\&id=u593e2bf1\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=319472\&status=done\&style=none\&taskId=u394249ff-dd6e-4f94-8a40-280cc16657a\&title=)

There are two approaches to support latching in a hash table that differ on the granularity of the latches:

* **Page Latches**: Each page has its own Reader-Writer latch that protects its entire contents. Threads acquire either a read or write latch before they access a page. This decreases parallelism because potentially only one thread can access a page at a time, but accessing multiple slots in a page will be fast for a single thread because it only has to acquire a single latch.
* **Slot Latches**: Each slot has its own latch. This increases parallelism because two threads can access different slots on the same page. But it increases the storage and computational overhead of accessing the table because threads have to acquire a latch for every slot they access, and each slot has to store data for the latches. The DBMS can use a single mode latch (i.e., Spin Latch) to reduce meta-data and computational overhead at the cost of some parallelism.

It is also possible to create a \*\*latch-free linear probing hash table \*\*directly using compare-and-swap (CAS) instructions. Insertion at a slot can be achieved by attempting to compare-and-swap a special “null” value with the tuple we wish to insert. If this fails, we can probe the next slot, continuing until it succeeds.

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371883853-9c770085-df36-40fc-bc83-29462d1a0a68.jpeg#averageHue=%23ececec\&clientId=u23110e91-7f27-4\&from=ui\&id=ucc51446b\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=316355\&status=done\&style=none\&taskId=ue2ad862b-4771-4295-bb0e-b5b5cb9d5a7\&title=)

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371884037-46db6735-e5a9-4419-bd65-69042f3e5b08.jpeg#averageHue=%23e9e9e9\&clientId=u23110e91-7f27-4\&from=ui\&id=u742e0e85\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=251531\&status=done\&style=none\&taskId=u6cef092a-c2cf-4898-bddf-c9235e60dcb\&title=)

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371884102-f30894b1-3638-453f-92e2-456e296e6c94.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u2452d4a2\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=278172\&status=done\&style=none\&taskId=u8c4ff6ce-0cee-4d9a-9a6f-223461e7234\&title=)

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371884397-cc2fad7b-c76c-44ba-a4c3-8f4514b3474d.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=uf7e2c0ef\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=278393\&status=done\&style=none\&taskId=ua6fe67a5-905d-4751-88fa-5cd58f5a9d1\&title=)

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371884472-8b8e073b-f2e3-44b0-8ffa-591b0a115dc9.jpeg#averageHue=%23e8e7e7\&clientId=u23110e91-7f27-4\&from=ui\&id=u70ead1ef\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=322650\&status=done\&style=none\&taskId=u0e9b67e3-b407-4c36-8ae6-4f34908bc9f\&title=)

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371885203-064a3a5d-ed6f-4cfe-a494-ad397f3df868.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u46f57f0c\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=281556\&status=done\&style=none\&taskId=u97182116-b6d3-48e8-85b0-da2c0dfd943\&title=)

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371885503-fe4ed175-17bd-43a1-b284-611c289803dc.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u9f8bef39\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=276054\&status=done\&style=none\&taskId=u7e547bd4-e94d-4aff-a486-4831caee1b4\&title=)

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371885585-8dc75476-a164-40d1-9ac6-028cc59e71c7.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=ua4efe0eb\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=275438\&status=done\&style=none\&taskId=uffb25a7d-0c28-4603-86c9-63fd317ad2f\&title=)

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371885791-bc847957-2739-4e95-b978-7ea414846505.jpeg#averageHue=%23e9e9e9\&clientId=u23110e91-7f27-4\&from=ui\&id=u5b35e17e\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=267457\&status=done\&style=none\&taskId=u6a7874ec-130f-44c7-b5f8-d2ae4f53a2f\&title=)

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371885906-6b9cd72a-9ccb-4043-b24d-627c04ed57e6.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=uc0181a40\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=272036\&status=done\&style=none\&taskId=u09e71cf7-baeb-4eca-a55e-5f5295e3f03\&title=)

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371886401-3cfd6230-6bf1-41d6-93bf-195b87136c61.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u0d4ca262\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=270369\&status=done\&style=none\&taskId=ua4f73a27-c969-4ed8-a888-8042242c048\&title=)

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371886688-3b8f4240-25a2-4e38-9377-272cce77f6d5.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u830f077e\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=274059\&status=done\&style=none\&taskId=uc981cb77-a8f1-4bf5-8e81-f263b0ca25c\&title=)

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371886827-681eb769-ffea-422f-af98-cec2c9b572d4.jpeg#averageHue=%23e9e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u433165dd\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=271286\&status=done\&style=none\&taskId=u305c3756-d5af-430d-9da0-b50cb8c5421\&title=)

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371887493-a5fd2a8e-1317-4f36-8459-4b06daf1c51a.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u345ce9e2\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=304429\&status=done\&style=none\&taskId=u3dd8e9f4-165c-43cb-a80e-e588da9dd57\&title=)

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371887545-990f7694-43b6-46ff-8b81-a901347984f1.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=ud6ca6faa\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=271382\&status=done\&style=none\&taskId=u7f66a0f6-3b3c-4c03-b2b2-83877835158\&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371887694-36de47f5-e8c8-470b-a0e8-35bdb3f9ff60.jpeg#averageHue=%23e9e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u5bd68640\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=273529\&status=done\&style=none\&taskId=u7c7ec06f-ae49-475c-87a2-d2d4c962c00\&title=)

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371888012-2c08570b-c129-47c0-9d2a-c31fa0ed8d42.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u5d560a91\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=272616\&status=done\&style=none\&taskId=ue7a58475-0e61-417b-8d1c-6d6be4359c1\&title=)

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371888173-b7ef200d-68e1-4c6f-ac0c-1addcdb0990d.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=uaa56b9f4\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=276028\&status=done\&style=none\&taskId=u43ef0cd1-543e-42d1-9890-a5924342ded\&title=)

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371889472-36ccc1eb-a389-4307-991e-a684e9e436c9.jpeg#averageHue=%23e8e8e8\&clientId=u23110e91-7f27-4\&from=ui\&id=u837e4eb0\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=276078\&status=done\&style=none\&taskId=u87205340-2093-455d-9ccf-1698e7ad63e\&title=)

## B+Tree Latching

The challenge of B+Tree latching is preventing the two following problems:

* Threads trying to modify the contents of a node at the same time.
* One thread traversing the tree while another thread splits/merges nodes.

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371889774-51b2c66a-6bf8-49de-b652-78c216e29bae.jpeg#averageHue=%23ececec\&clientId=u23110e91-7f27-4\&from=ui\&id=udd20a6e3\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=312033\&status=done\&style=none\&taskId=u6573bd10-8b0c-41b0-a683-74e2a3d15eb\&title=)

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371889805-39f89e4a-f1f6-4bed-8ef3-7e72333ccb02.jpeg#averageHue=%23eaeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u9a65f6b3\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=297652\&status=done\&style=none\&taskId=uf46459b1-d40a-4f88-8a6a-752d08e9538\&title=)

![37.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371890172-c4356270-85b3-40bc-8637-8ddc24f84b41.jpeg#averageHue=%23eae9e9\&clientId=u23110e91-7f27-4\&from=ui\&id=uf059023c\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=309768\&status=done\&style=none\&taskId=u08c40a12-3fbd-4600-a577-0ad600e0da6\&title=)

![38.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371890277-042eee56-43d8-455d-ac28-d7bcc7382540.jpeg#averageHue=%23eae9e9\&clientId=u23110e91-7f27-4\&from=ui\&id=uceb9a00c\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=322336\&status=done\&style=none\&taskId=u625d150b-863c-40e0-9dad-301fd9af151\&title=)

![39.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371890871-df6ec0bc-64d2-4963-b3ad-138b821fb4ed.jpeg#averageHue=%23eaeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u4171837b\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=323331\&status=done\&style=none\&taskId=uaf1e2a29-1d21-4e92-b2a0-f08393979da\&title=)

![40.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371891281-356facde-9abd-44d5-9c6b-4690b4eb5535.jpeg#averageHue=%23eaeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u3fa17dbb\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=312612\&status=done\&style=none\&taskId=ua2e41c56-1875-41ba-abaf-65403e8adaf\&title=)

![41.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371891390-d2743504-b9e0-4d6b-b9b4-8c9afaee5c09.jpeg#averageHue=%23eaeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=ue1dddd17\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=311768\&status=done\&style=none\&taskId=ufb500bb4-703b-4a6f-a599-992b595cf47\&title=)

Latch crabbing/coupling is a protocol to allow multiple threads to access/modify B+Tree at the same time. The basic idea is as follows.

1. Get latch for the parent.
2. Get latch for the child.
3. Release latch for the parent **if the child is deemed “safe”** . A “safe” node is one that will not split, merge, or redistribute when updated.

Note that the notion of “safe” depends on whether the operation is an insertion or a deletion. A full node is “safe” for deletion since a merge will not be needed but is not “safe” for an insertion since we may need to split the node. Note that read latches do not need to worry about the “safe” condition.

![42.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371891799-3c9bbd30-d9e5-4893-b5e8-7d714ffac836.jpeg#averageHue=%23ececec\&clientId=u23110e91-7f27-4\&from=ui\&id=u72bc4007\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=312282\&status=done\&style=none\&taskId=ua2258acf-2b56-4bac-85a9-e5960c17231\&title=)

### Basic Latch Crabbing Protocol:

* \*\*Search: \*\*Start at the root and go down, repeatedly acquire latch on the child and then unlatch parent.
* **Insert/Delete:** Start at the root and go down, obtaining `X` latches as needed. Once the child is latched, check if it is safe. If the child is safe, release latches on all its ancestors.

The order in which latches are released is not important from a correctness perspective. However, from a performance point of view, it is better to release the latches that are higher up in the tree since they block access to a larger portion of leaf nodes.

![43.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371891877-4c7f1945-b310-43f4-9a0e-cc7cf5facac1.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=uaa582930\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=319622\&status=done\&style=none\&taskId=ue8d57339-1607-43ca-b9a6-54586205d1f\&title=)

![44.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371892544-35a7a3a3-cd07-4a92-86ff-d602eb5460e0.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u8f161889\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=279010\&status=done\&style=none\&taskId=uea3e2df2-6b7e-49f1-9535-a89c7f04695\&title=)

![45.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371892930-e6649b8e-a853-4918-be1b-1c545e6ee6a9.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u25b97576\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=317803\&status=done\&style=none\&taskId=u0423b995-1cf8-4755-98c4-d257048b568\&title=)

![46.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371893238-ed641d9c-3125-44db-b8be-6a5763eb78bb.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=uedb511e7\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=278933\&status=done\&style=none\&taskId=uaaf862fb-38ed-4308-b676-cc75eb85c8d\&title=)

![47.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371893319-60b31228-46a5-4ecc-8292-4b6804de21cb.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u03a71d8c\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=277802\&status=done\&style=none\&taskId=u96d60dae-7ea5-4aec-aaf0-3415b29cea5\&title=)

![48.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371893365-1a2b30a6-47f3-4e42-9a46-b9bfc564c8ae.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u4ae7adbb\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=273387\&status=done\&style=none\&taskId=u95f06b2f-480f-4f30-b052-63494feb3bf\&title=)

![49.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371894228-d04507b5-e415-4693-bdd4-f55a0effc5c2.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=uab43bb6c\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=280954\&status=done\&style=none\&taskId=ucf98c8b0-ee25-4719-a8ac-a60d749945a\&title=)

![50.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371894488-dc131a6c-63a5-47fc-bfef-19ab8a0801b3.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=ucb9cfab3\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=333417\&status=done\&style=none\&taskId=uc28aeb4d-16b5-4f4b-9e5b-fbb0d52d49e\&title=)

![51.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371894585-c09f7de9-c97e-4c63-a450-ed403b340ca8.jpeg#averageHue=%23ecebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u59b5a473\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=339240\&status=done\&style=none\&taskId=uc42ab7e6-8a22-4035-bb62-7d86668171b\&title=)

![52.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371894918-06948810-f664-4634-8c88-1fa7240c84df.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=ua478da8d\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=327065\&status=done\&style=none\&taskId=ub571f280-28d0-4c12-b081-feb4d22978e\&title=)

![53.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371894989-ff050570-cd8e-4cd7-8e80-c82c811d3c43.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u06b26451\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=279709\&status=done\&style=none\&taskId=u85171e65-cd74-40a5-97f1-426d006f0f5\&title=)

![54.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371896134-258b2e25-225c-431f-aece-6284784b825a.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=ue1d3ff65\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=349150\&status=done\&style=none\&taskId=ucac09658-a4db-4469-86a7-134d048b6cf\&title=)

![55.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371896339-8e30f1e1-f4aa-43b9-bdbf-a1f2ca7f9c25.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u014be203\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=286648\&status=done\&style=none\&taskId=uad43cce6-1e50-4859-a001-7605971411f\&title=)

![56.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371896428-21fba36e-0266-418d-ab96-c9bb7eeafd43.jpeg#averageHue=%23eaeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u7b9e4c19\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=322366\&status=done\&style=none\&taskId=u5642ded7-bde4-4153-8ac9-103445ed33f\&title=)

![57.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371896454-29240dea-6931-4d8e-aac0-c98c001d5db4.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u9f7cadbe\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=286661\&status=done\&style=none\&taskId=u97303adc-9796-4466-a315-f78eed44aa6\&title=)

![58.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371896523-b5d1fbf6-5059-44d9-b764-123cf1daab7a.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u4147f269\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=280088\&status=done\&style=none\&taskId=ub383169a-2cd6-43c8-91fe-f24b48c9d3a\&title=)

![59.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371897429-77a570dd-d0c3-4906-a6b8-6a965c91257f.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u901e68e6\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=280293\&status=done\&style=none\&taskId=ub94f4bf2-6be7-4dd6-8c42-efa683e343d\&title=)

![60.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371897909-c80c2881-489f-4b65-88ae-c481407e475c.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=ub31715f6\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=342030\&status=done\&style=none\&taskId=u0ff5c621-913c-46cb-a0fe-e8644134fc3\&title=)

![61.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371898017-02bfd225-018e-4b6d-b4fe-bb0f80b42a77.jpeg#averageHue=%23eaeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u05bde0d0\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=352067\&status=done\&style=none\&taskId=u1ae767b9-5eb7-4cef-b7f6-d97423eb57e\&title=)

### Improved Latch Crabbing Protocol:

The problem with the basic latch crabbing algorithm is that transactions always acquire an exclusive latch on the root for every insert/delete operation. This limits parallelism. Instead, one can assume that having to resize (i.e., split/merge nodes) is rare, and thus transactions can acquire shared latches down to the leaf nodes. Each transaction will assume that the path to the target leaf node is safe, and use `READ` latches and crabbing to reach it and verify. If the leaf node is not safe, then we abort and do the previous algorithm where we acquire `WRITE` latches.

![62.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371898093-d8efcbdf-d176-499a-8c74-104c554e1cca.jpeg#averageHue=%23eaeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u43b9dca5\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=362722\&status=done\&style=none\&taskId=u8197edca-e645-441e-ab11-4a1b7e43074\&title=)

* **Search:** Same algorithm as before.
* \*\*Insert/Delete: \*\*Set `READ` latches as if for search, go to leaf, and set `WRITE` latch on leaf. If the leaf is not safe, release all previous latches, and restart the transaction using previous Insert/Delete protocol.

![63.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371898186-aa11a0be-594e-46bf-bcb1-b7841420ba94.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u007373d4\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=514561\&status=done\&style=none\&taskId=u22b31d9c-bc87-4d09-8b2e-c0a1059052e\&title=)

![64.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371899088-def6e57f-da32-471d-aad7-0e2d580c2df1.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=ue66307fa\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=354187\&status=done\&style=none\&taskId=u2647f716-299e-4d0c-bb43-497a88677e5\&title=)

![65.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371899555-9acf990e-d8c8-4cce-a286-f2fc0df7cbc5.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=ueec3868f\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=280777\&status=done\&style=none\&taskId=u016bbdae-3a61-48c9-9c30-902b31a0df4\&title=)

![66.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371899683-a49aeb4c-756e-4a45-be1d-c4a745c29fc2.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=ue05ad772\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=279698\&status=done\&style=none\&taskId=uea7537d2-faf4-4976-aa08-b1ee790bac3\&title=)

![67.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371899681-06c95b6f-126e-4d19-8f23-7101dd9b0a4d.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=ucd9b07c5\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=280732\&status=done\&style=none\&taskId=u4bbb752e-ef7d-4b7b-9f4e-b9ea174090f\&title=)

![68.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371900167-db680c0c-e58e-4f13-86ac-fbd02dd917f8.jpeg#averageHue=%23ebeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u36b18412\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=313783\&status=done\&style=none\&taskId=ue76b4381-507f-45c2-9eba-40d2d4bc182\&title=)

![69.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371900640-93124993-e9a5-4ede-888d-a662f22233d8.jpeg#averageHue=%23eaeaea\&clientId=u23110e91-7f27-4\&from=ui\&id=u28ec13f5\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=307697\&status=done\&style=none\&taskId=u36a0b53a-8b7a-40d2-80f3-7c7f5ccf404\&title=)

![70.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371900917-2264e1ff-9c1e-45af-b8e4-7fad6bc29a70.jpeg#averageHue=%23ebebeb\&clientId=u23110e91-7f27-4\&from=ui\&id=u3672c67e\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=329346\&status=done\&style=none\&taskId=u765fb34e-3155-4ea7-be4a-c670a730e49\&title=)

## Leaf Node Scans

The threads in these protocols acquire latches in a “top-down” manner. This means that a thread can only acquire a latch from a node that is below its current node. If the desired latch is unavailable, the thread must wait until it becomes available. Given this, there can never be deadlocks. However, **leaf node scans are susceptible to deadlocks** because now we have threads trying to acquire exclusive locks in two different directions at the same time (e.g., thread 1 tries to delete, thread 2 does a leaf node scan). **Index latches do not support deadlock detection or avoidance**. Thus, the only way programmers can deal with this problem is **through coding discipline**. The leaf node sibling latch acquisition protocol must support a “no-wait” mode. That is, the B+tree code must cope with failed latch acquisitions. Since latches are intended to be held (relatively) brieﬂy, if a thread tries to acquire a latch on a leaf node but that latch is unavailable, then it should abort its operation (releasing any latches that it holds) quickly and restart the operation.

![71.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371901204-5b562eec-dfc3-41bc-bc9a-ea4db980f11e.jpeg#averageHue=%23ececec\&clientId=u23110e91-7f27-4\&from=ui\&id=uaae78859\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=322517\&status=done\&style=none\&taskId=u08f83bab-2a07-4005-a4ac-6df12d2498c\&title=)

![72.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371901179-2bde69e2-3fea-41cc-86b8-623d4777976d.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u63bf6754\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=206359\&status=done\&style=none\&taskId=u0b2d84c9-7be5-4c78-8aa1-fe47f24fff1\&title=)

![73.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371901678-9932ae72-0ffd-4e4e-8e73-5a5fe52aa7a1.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u4fd6ea09\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=204883\&status=done\&style=none\&taskId=u2bb4222d-ec33-403d-a035-e1b80dc7b8a\&title=)

![74.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371902278-de6538c5-fbde-4a39-b3f5-a5cc1162a9b9.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u8b59b510\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=256649\&status=done\&style=none\&taskId=u1629d595-2332-467c-9e37-8d16a67cdc1\&title=)

![75.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371902271-54f225cf-11e7-40bf-b649-8f35a1062bfd.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u393b9037\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=206682\&status=done\&style=none\&taskId=u2d9a6398-8cbb-48cf-8437-06f1de34198\&title=)

![76.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371902418-640e82a4-ee83-4fcd-81cf-fe659a5a7db3.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u141d62e7\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=225988\&status=done\&style=none\&taskId=u7f395e3b-1dab-40f5-aef4-adbd2f160b1\&title=)

![77.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371902603-5b1d76f7-bca8-4af5-985d-4103bb7d0d72.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u7aa87e43\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=239067\&status=done\&style=none\&taskId=u1b9be989-5414-4df0-b45f-46e154d5576\&title=)

![78.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371902923-ff085697-fe0b-49fb-ac66-374ab1386442.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u1445361d\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=231996\&status=done\&style=none\&taskId=ua53627fe-eaf3-495e-b5f1-155e3cf3c31\&title=)

![79.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371903595-6e980d1d-7058-4e8a-a2d2-4823c786d627.jpeg#averageHue=%23eeeded\&clientId=u23110e91-7f27-4\&from=ui\&id=ueb922273\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=305337\&status=done\&style=none\&taskId=u40d78bca-8b7f-4d9e-a5ed-6b9372fa271\&title=)

![80.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371904225-def4884f-60f7-46e0-94cc-7b12312de095.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=ua4d6e29d\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=288950\&status=done\&style=none\&taskId=ub0653b2a-858f-4e09-9166-357c3bf153d\&title=)

![81.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371904178-8f1867d1-c96f-40ce-a386-846c5ad044a4.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u0eb1e9c3\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=213432\&status=done\&style=none\&taskId=u90c5c569-0945-4ad3-bb0e-7d507b66ba9\&title=)

![82.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371904255-0e85cce9-002f-4884-97f1-e4dc127bd183.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=ud1bf2309\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=220544\&status=done\&style=none\&taskId=u581fd517-d951-4cf4-a1df-c71f245e038\&title=)

![83.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371904406-9763eabb-d32e-4e15-b76f-9fac03743ca2.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u41f97199\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=230079\&status=done\&style=none\&taskId=u58a8944a-fc5a-4799-9d0a-ead82ff71d6\&title=)

![84.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371905047-a50c7975-1ce7-4f41-9a03-06150a1c41c4.jpeg#averageHue=%23eeeded\&clientId=u23110e91-7f27-4\&from=ui\&id=u13d4ed89\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=296051\&status=done\&style=none\&taskId=u7dcbac4c-c2dd-4059-93af-d7b8aed36c5\&title=)

![85.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371905824-e41d847d-54e0-4f8d-9010-a05b843ef8a2.jpeg#averageHue=%23ececec\&clientId=u23110e91-7f27-4\&from=ui\&id=ued42dd9d\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=302721\&status=done\&style=none\&taskId=u4482b390-bff1-47c1-9a91-4d18eeb8fbb\&title=)

![86.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371905871-0dfa7432-6dd7-4b6a-a026-abee7894514d.jpeg#averageHue=%23ececec\&clientId=u23110e91-7f27-4\&from=ui\&id=u96b5b174\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=326917\&status=done\&style=none\&taskId=uca081312-df12-4e21-a9cb-c76a480a1a4\&title=)

![87.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371905865-92aff0d0-0392-4e53-b8d5-ccd1374d8fdb.jpeg#averageHue=%23eeeeee\&clientId=u23110e91-7f27-4\&from=ui\&id=u30696a65\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=247406\&status=done\&style=none\&taskId=ua9597975-320a-49d6-8198-50d7cbad634\&title=)

![88.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678371905859-4b5618ab-44fa-42df-8587-24586bb11c51.jpeg#averageHue=%23f0f0f0\&clientId=u23110e91-7f27-4\&from=ui\&id=udca81c1a\&originHeight=1688\&originWidth=3000\&originalType=binary\&ratio=2\&rotation=0\&showTitle=false\&size=172005\&status=done\&style=none\&taskId=u85d38998-3119-498a-9291-a01780d960c\&title=)
