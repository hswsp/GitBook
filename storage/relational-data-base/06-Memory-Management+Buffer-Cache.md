# 06 - Memory Management + Buffer Cache

![1.jpg](assets/1678153348255-1c9f91c4-0686-44db-979c-b1349e10ac2e.jpeg)

![2.jpg](assets/1678153347656-7748c4f5-a636-46f1-abeb-56f5b49cb1d3.jpeg)

![3.jpg](assets/1678153347676-3b83b2e0-688a-490f-a4e6-98269058df81.jpeg)

# Introduction

The DBMS is responsible for managing its memory and moving data back-and-forth from the disk. Since, for the most part, data cannot be directly operated on in the disk, any database must be able to efficiently move data represented as files on its disk into memory so that it can be used. A diagram of this interaction is shown in Figure 1. A obstacle that DBMS’s face is the problem of minimizing the slowdown of moving data around. Ideally, it should “appear” as if the data is all in the memory already. The **execution engine** shouldn’t have to worry about how data is fetched into memory.

![Figure 1: Disk-oriented DBMS.](assets/image.png)

Another way to think of this problem is in terms of spatial and temporal control.
*Spatial Control* refers to where pages are physically written on disk. The goal of spatial control is to keep pages that are used together often as physically close together as possible on disk.
*Temporal Control* refers to when to read pages into memory and when to write them to disk. Temporal control aims to minimize the number of stalls from having to read data from disk.

![4.jpg](assets/20240307174307.jpeg)

![5.jpg](assets/1678153347864-897dc818-d574-4cc7-b28c-454cb31f5100.jpeg)

![6.jpg](assets/1678153349614-1b38927b-8de5-4923-ad38-c1efa831f77b.jpeg)

![7.jpg](assets/1678153349416-73812439-0f41-444f-90c3-14eb03f78c36.jpeg)

# Locks vs. Latches

We need to make a distinction between locks and latches when discussing how the DBMS protects its internal elements.
**Locks: **A lock is a higher-level, logical primitive that protects the contents of a database (e.g., tuples, tables, databases) from other transactions. Transactions will hold a lock for its entire duration. Database systems can expose to the user which locks are being held as queries are run. Locks need to be able to rollback changes.
**Latches:**  A latch is a low-level protection primitive that the DBMS uses for the critical sections in its internal data structures (e.g., hash tables, regions of memory). Latches are held for only the duration of the operation being made. Latches do not need to be able to rollback changes.

# Buffer Pool

The *buffer pool* is an in-memory cache of pages read from disk. It is essentially a large memory region allocated inside of the database to store pages that are fetched from disk.
The buffer pool’s region of memory organized as an array of **fixed size pages.**  Each array entry is called a **frame**. When the DBMS requests a page, an exact copy is placed into one of the frames of the buffer pool. Then, the database system can search the buffer pool first when a page is requested. If the page is not found, then the system fetches a copy of the page from the disk. Dirty pages are buffered and not written back immediately（write-back cache）. See Figure 2 for a diagram of the buffer pool’s memory organization.

![Figure 2: Buffer pool organization and meta-data](assets/image1.png)

![8.jpg](assets/1678153349892-778ced66-d7d6-4c13-a231-72e2bc312552.jpeg)

![9.jpg](assets/1678153350320-101aa9ff-3973-4339-9fb6-d66d3f99ee83.jpeg)

## Buffer Pool Meta-data

The buffer pool must maintain certain meta-data in order to be used efficiently and correctly.
Firstly, the *page table* is an in-memory hash table that keeps track of pages that are currently in memory. It maps page ids to frame locations in the buffer pool. Since the order of pages in the buffer pool does not necessarily reflect the order on the disk, this extra indirection layer allows for the identification of page locations in the pool.
**Note: **The **page table** is not to be confused with the **page directory**, which is the mapping from page ids to page locations in database files. All changes to the page directory must be recorded on disk to allow the DBMS to find on restart.
The page table also maintains additional meta-data per page, a dirty-flag and a pin/reference counter.

![10.jpg](assets/1678153350463-317bf974-8dbf-4f19-9c65-3f7e5e3ffe10.jpeg)

The *dirty-flag* is set by a thread whenever it modifies a page. This indicates to storage manager that the page must be written back to disk.
The *pin/reference* Counter tracks the number of threads that are currently accessing that page (either reading or modifying it). A thread has to increment the counter before they access the page. If a page’s count is greater than zero, then the storage manager is **not** allowed to evict that page from memory.

![11.jpg](assets/1678153351043-762701db-a635-4bf1-9a0d-d3037dcfa0e0.jpeg)

![12.jpg](assets/1678153351606-2ef93a63-9973-4297-84e1-b3a71bb69576.jpeg)

![13.jpg](assets/1678153352400-c5911648-e775-4348-b72c-9cf0159dc982.jpeg)

![14.jpg](assets/1678153352763-ac24cc40-bb62-4b3e-8144-2ba3915217e5.jpeg)

## Memory Allocation Policies

Memory in the database is allocated for the buffer pool according to two policies.
*Global policies* deal with decisions that the DBMS should make to benefit the entire workload that is being executed. It considers all active transactions to find an optimal decision for allocating memory.
An alternative is *local policies*, which makes decisions that will make a single query or transaction run faster, even if it isn’t good for the entire workload. Local policies allocate frames to a specific transactions without considering the behavior of concurrent transactions.
Most systems use a combination of both global and local views.

![15.jpg](assets/1678153352564-53946bc9-b383-451e-9fd6-4673dc9cb7ff.jpeg)

# Buffer Pool Optimizations

There are a number of ways to optimize a buffer pool to tailor it to the application’s workload.

![16.jpg](assets/1678153353126-1d179913-7b01-449b-8c47-3ea6b1db78f0.jpeg)

## Multiple Buffer Pools

The DBMS can maintain multiple buffer pools for different purposes (i.e per-database buffer pool, per-page type buffer pool). Then, each buffer pool can adopt local policies tailored for the data stored inside of it. This method can help reduce latch contention and improves locality.
Two approaches to mapping desired pages to a buffer pool are object IDs and hashing.
_Object IDs _involve extending the record IDs to have an object identifier. Then through the object identifier, a mapping from objects to specific buffer pools can be maintained.
Another approach is *hashing* where the DBMS hashes the page id to select which buffer pool to access.

![17.jpg](assets/1678153353595-ec002489-d228-4595-89f8-21c00752c522.jpeg)

bufferpool -> tablespace -> table

![18.jpg](assets/1678153354566-eb795e51-4623-4624-93ac-0d40380784dc.jpeg)

![19.jpg](assets/1678153355297-d7e24410-688d-4e10-9f87-ca03b40203ba.jpeg)

![20.jpg](assets/1678153355297-0567d3a4-9794-42fb-96bf-de6365c3d931.jpeg)

## Pre-fetching

The DBMS can also optimize by pre-fetching pages based on the query plan. Then, while the first set of pages is being processed, the second can be pre-fetched into the buffer pool. This method is commonly used by DBMS’s when accessing many pages sequentially.

![21.jpg](assets/1678153355238-8ee7a04f-cb67-4f70-a7cc-a43b9f28dd0a.jpeg)

![22.jpg](assets/1678153355717-af80b53f-35c5-4b52-93c1-bbe89f56b696.jpeg)

![23.jpg](assets/1678153356804-9af370fe-bd9c-4449-9c4d-115b9e2ebf57.jpeg)

![24.jpg](assets/1678153357138-773eb9ee-b908-4b67-a404-c969bcfcdf7c.jpeg)

![25.jpg](assets/1678153357336-522c5bd9-d26c-42e8-8d6b-e2b4d758bab4.jpeg)

![26.jpg](assets/1678153357355-8e9e2118-27b2-49a9-972f-cbafaf3765d7.jpeg)

![27.jpg](assets/1678153357676-ede328bb-a3ba-4bba-b332-8b89166ddc77.jpeg)

![28.jpg](assets/1678153359034-12bc18f5-3768-4ec1-8ef1-9a12ce4679bd.jpeg)

![29.jpg](assets/1678153359481-6c34b9b6-f8d0-4723-9dbb-16d38526aae1.jpeg)

## Scan Sharing (Synchronized Scans)

Query cursors can reuse data retrieved from storage or operator computations. This allows multiple queries to attach to a single cursor that scans a table. Ifb a query starts a scan and if there one already doing this, then the DBMS will attach the second query’s cursor to the existing cursor. The DBMS keeps track of where the second query joined with the first so that it can finish the scan when it reaches the end of the data structure.

![30.jpg](assets/1678153359291-97da5221-c917-4c04-98d7-3ca13bccc931.jpeg)

![31.jpg](assets/1678153359453-6ea90766-f180-497e-a793-ade446fc481c.jpeg)

Problems that Synchronized Scans might incur:

![32.jpg](assets/1678153360184-39378148-2528-4212-a6d7-534d9c8ffe12.jpeg)

![33.jpg](assets/1678153361530-cc02527a-ae9b-4867-9452-790a11e441fb.jpeg)

![34.jpg](assets/1678153361130-e5b0a992-4db5-4528-9df3-b1d6cd9a3105.jpeg)

![35.jpg](assets/1678153361335-5f43dc4f-10da-47f5-bef2-7d0ac3c8cd55.jpeg)

![36.jpg](assets/1678153361563-08a742c6-25d6-4691-9ab2-c509817c886e.jpeg)

![37.jpg](assets/1678153362073-e93a65d1-41a5-468c-9f2c-ed812694dae7.jpeg)

![38.jpg](assets/1678153363064-808cf23a-11ca-4a44-941c-8fd5a5d64813.jpeg)

![39.jpg](assets/1678153363587-d54260b6-49bf-4aa8-9c52-40de1c584417.jpeg)

![40.jpg](assets/1678153363599-8856ce13-a3bb-44da-91c1-c1b981dce7dc.jpeg)

![41.jpg](assets/1678153364058-00543f1d-7a9d-4652-b81e-c978e7b2ab27.jpeg)

![42.jpg](assets/1678153364063-47a633ca-277b-42f9-9b35-97a23061655d.jpeg)

![43.jpg](assets/1678153365427-10d63e1b-02b4-4ded-8253-4a29d1ba3993.jpeg)

![44.jpg](assets/1678153365529-f6531f67-a7f0-4a59-83ef-3601d500c48c.jpeg)

## Buffer Pool Bypass

The sequential scan operator will not store fetched pages in the buffer pool to avoid overhead. Instead, memory is local to the running query. This works well if operator needs to read a large sequence of pages that are contiguous on disk. Buffer Pool Bypass can also be used for temporary data (sorting, joins).

![45.jpg](assets/1678153365937-37718a9f-8ebf-4e5d-bca9-b11767245341.jpeg)

# OS Page Cache

Most disk operations go through the OS API. Unless explicitly told otherwise, the OS maintains its own filesystem cache.
Most DBMS use **direct I/O to bypass the OS’s cache** in order to avoid redundant copies of pages and having to manage different eviction policies.
**Postgres** is an example of a database system that uses the OS’s Page Cache.

![46.jpg](assets/1678153366172-21fe8fa4-79d2-4867-b753-54eb41b1b48e.jpeg)

![47.jpg](assets/1678153366285-12dc6b27-a955-4ea6-bf0c-2fd1c4da8a22.jpeg)

![48.jpg](assets/1678153367333-85e31de8-2df6-4dcc-936e-b973d4badd0b.jpeg)

# Buffer Replacement Policies

When the DBMS needs to free up a frame to make room for a new page, it must decide which page to evict from the buffer pool.
A replacement policy is an algorithm that the DBMS implements that makes a decision on which pages to evict from buffer pool when it needs space.
Implementation goals of replacement policies are improved correctness, accuracy, speed, and meta-data overhead.

![49.jpg](assets/1678153367319-d6da88b3-a735-4cac-97b0-5cd1f3b93cce.jpeg)

## Least Recently Used (LRU)

The Least Recently Used replacement policy maintains a timestamp of when each page was last accessed. The DBMS picks to evict the page with the oldest timestamp. This timestamp can be stored in a separate data structure, such as a queue, to allow for sorting and improve efficiency by reducing sort time on eviction.

![50.jpg](assets/1678153367993-ea1e1b02-6d91-445f-81f6-c6c80a8e79e5.jpeg)

## CLOCK

The CLOCK policy is an approximation of LRU without needing a separate timestamp per page. In the CLOCK policy, each page is given a **reference bit**. When a page is accessed, set to 1.
To visualize this, organize the pages in a circular buffer with a “clock hand”. Upon sweeping check if a page’s bit is set to 1. If yes, set to zero, if no, then evict it. In this way, the clock hand remembers position between evictions.

![Figure 3: Visualization of CLOCK replacement policy. Page 1 is referenced and set to 1. When the clock hand sweeps, it sets the reference bit for page 1 to 0 and evicts page 5.](assets/image2.png)

![51.jpg](assets/1678153368468-f72a04a3-8710-4884-9de1-efe86bb28118.jpeg)

![52.jpg](assets/1678153368635-1d42ab91-85ab-4412-bb26-3e76dd4f0f3b.jpeg)

![53.jpg](assets/1678153369195-35cc5ba2-ff5f-43ba-ad89-15489b88f494.jpeg)

![54.jpg](assets/1678153369471-11590e4b-677a-4507-ae94-22e155ba9b3f.jpeg)

![55.jpg](assets/1678153370533-65e8af5a-74fa-4871-946d-74bb08a8850e.jpeg)

![56.jpg](assets/1678153370803-1bee71cf-bc9f-448c-9605-de60081425a8.jpeg)

![57.jpg](assets/1678153370790-7b94cbe7-e8c9-44d9-a3bf-9348ea37c3a8.jpeg)

![58.jpg](assets/1678153371304-3de34c04-dffd-418f-a229-e975e2ab13ef.jpeg)

![59.jpg](assets/1678153371404-6125964d-c5cd-4867-8445-dcc6db05b0f2.jpeg)

![60.jpg](assets/1678153372689-abd86d66-e99e-45c8-9099-8fc25bdc323c.jpeg)

![61.jpg](assets/1678153372919-6ae9f3d8-85dd-4fa3-9ca3-33750266b766.jpeg)

![62.jpg](assets/1678153373013-3e0652db-fcb4-4771-90dd-2054e413f0d6.jpeg)

![63.jpg](assets/1678153373455-156aec37-003f-4178-8e6d-c9204d10d6c9.jpeg)

![64.jpg](assets/1678153373560-eeddeb65-4104-47ec-9a46-3c6c5ef91fbf.jpeg)

![65.jpg](assets/1678153374750-e1cfdc0d-352c-43c5-8396-6daa81d0b40a.jpeg)

## Alternatives

There are a number of problems with LRU and CLOCK replacement policies.
Namely, LRU and CLOCK are susceptible to *sequential flooding*, where the buffer pool’s contents are corrupted due to a sequential scan. Since sequential scans read every page, the timestamps of pages read may not reflect which pages we actually want. In other words, the most recently used page is actually the most unneeded page.
There are three solutions to address the shortcomings of LRU and CLOCK policies.
One solution is **LRU-K **which tracks the history of the last K references as timestamps and computes the interval between subsequent accesses. This history is used to predict the next time a page is going to be accessed.

![66.jpg](assets/1678153374904-d60efdd3-0a76-4670-80a2-f8489f78535d.jpeg)

Another optimization is localization per query. The DBMS chooses which pages to evict on a per transaction/query basis. This minimizes the pollution of the buffer pool from each query.

![67.jpg](assets/1678153374840-4491b5e2-794f-43c0-8a51-b5bcf4ad65cf.jpeg)

Lastly, priority hints allow transactions to tell the buffer pool whether page is important or not based on the context of each page during query execution.

![68.jpg](assets/1678153375485-b8bb1a35-ea92-4cf5-bf7b-0c249a429ef5.jpeg)

![69.jpg](assets/1678153375609-802cfdfb-80d5-4e5a-86cc-e400e58414d2.jpeg)

## Dirty Pages

There are two methods to handling pages with dirty bits. The fastest option is to drop any page in the buffer pool that is not dirty. A slower method is to write back dirty pages to disk to ensure that its changes are persisted.
These two methods illustrate the trade-off between fast evictions versus dirty writing pages that will not be read again in the future.

![70.jpg](assets/1678153377123-995eee76-db12-4c9d-a031-89199343e936.jpeg)

One way to avoid the problem of having to write out pages unnecessarily is background writing. Through background writing, the DBMS can periodically walk through the page table and write dirty pages to disk. When a dirty page is safely written, the DBMS can either evict the page or just unset the dirty flag.

![71.jpg](assets/1678153377211-404b5116-5d2f-4f80-96a7-9e6699791968.jpeg)

# Other Memory Pools

The DBMS needs memory for things other than just tuples and indexes. These other memory pools may not always backed by disk depending on implementation.

- Sorting + Join Buffers
- Query Caches
- Maintenance Buffers
- Log Buffers
- Dictionary Caches

![72.jpg](assets/1678153377213-e068ce30-83b5-4ff3-b23a-6f99fe9e68db.jpeg)

![73.jpg](assets/1678153377511-24127ce2-d558-4097-b250-bb17f6e31f9c.jpeg)

![74.jpg](assets/1678153377406-e444126d-c91d-41cb-af4b-637f317aa960.jpeg)

![75.jpg](assets/1678153379374-a3b3da12-cce1-4ebf-ae13-a50cc33c2a9a.jpeg)

![76.jpg](assets/1678153379272-0072a04e-713e-4581-8ef0-26500494c01a.jpeg)

![77.jpg](assets/1678153379357-a9132a1f-5bdd-481b-a00b-029498dcb7f8.jpeg)

![78.jpg](assets/1678153379604-77e68d5f-5885-42ae-ac37-0e0da4809c22.jpeg)

![79.jpg](assets/1678153379376-c4afc67a-0f69-48d0-8792-236224107d41.jpeg)

![80.jpg](assets/1678153381589-0e69fc7e-127a-4f8e-9560-3e031ef0c8c4.jpeg)

![81.jpg](assets/1678153381728-157fa72a-575a-4bc3-8b06-2362d154a09d.jpeg)

![82.jpg](assets/1678153381851-571cd62f-8879-423f-bd80-929b95f3364b.jpeg)
