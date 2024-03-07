# 06 - Memory Management + Buffer Cache

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153348255-1c9f91c4-0686-44db-979c-b1349e10ac2e.jpeg#averageHue=%231d2635&clientId=uae7a86c3-043f-4&from=ui&id=u65efeb0e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=597819&status=done&style=none&taskId=u3f905d2f-4e71-4887-92f6-da5db3bff40&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153347656-7748c4f5-a636-46f1-abeb-56f5b49cb1d3.jpeg#averageHue=%23eeeeee&clientId=uae7a86c3-043f-4&from=ui&id=u41a6f812&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=264090&status=done&style=none&taskId=u656ca483-9e2b-43cb-bcb1-15f02834cfc&title=)

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153347676-3b83b2e0-688a-490f-a4e6-98269058df81.jpeg#averageHue=%23ededed&clientId=uae7a86c3-043f-4&from=ui&id=u2d1f48d6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=279990&status=done&style=none&taskId=uc8f36bd6-47d5-4518-be55-cebc135060e&title=)

# Introduction

The DBMS is responsible for managing its memory and moving data back-and-forth from the disk. Since, for the most part, data cannot be directly operated on in the disk, any database must be able to efficiently move data represented as files on its disk into memory so that it can be used. A diagram of this interaction is shown in Figure 1. A obstacle that DBMS’s face is the problem of minimizing the slowdown of moving data around. Ideally, it should “appear” as if the data is all in the memory already. The **execution engine** shouldn’t have to worry about how data is fetched into memory.

![Figure 1: Disk-oriented DBMS.](https://cdn.nlark.com/yuque/0/2023/png/22382307/1678153973572-7ed2c0dc-91f1-42bb-a8e4-0d69329c47c3.png#averageHue=%23f2f0f0&clientId=uae7a86c3-043f-4&from=paste&height=456&id=u5ef85e69&originHeight=912&originWidth=1998&originalType=binary&ratio=2&rotation=0&showTitle=true&size=168684&status=done&style=none&taskId=u20fcfef1-7ab5-48e2-94c5-4df99480ae2&title=Figure%201%3A%20Disk-oriented%20DBMS.&width=999)

Another way to think of this problem is in terms of spatial and temporal control.
*Spatial Control* refers to where pages are physically written on disk. The goal of spatial control is to keep pages that are used together often as physically close together as possible on disk.
*Temporal Control* refers to when to read pages into memory and when to write them to disk. Temporal control aims to minimize the number of stalls from having to read data from disk.

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153347699-8d798579-c649-4ec6-9afe-5d34c7475c37.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=uff44c26b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319265&status=done&style=none&taskId=u7adbeb7d-fd74-4666-b029-059016dce94&title=)

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153347864-897dc818-d574-4cc7-b28c-454cb31f5100.jpeg#averageHue=%23e7e6e6&clientId=uae7a86c3-043f-4&from=ui&id=ua4bbafcb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=365206&status=done&style=none&taskId=u70f05be9-9b52-4ca4-9bbe-9ebc595ce38&title=)

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153349614-1b38927b-8de5-4923-ad38-c1efa831f77b.jpeg#averageHue=%23e7e6e6&clientId=uae7a86c3-043f-4&from=ui&id=u74965430&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=343409&status=done&style=none&taskId=u793c7736-a6dd-4206-949e-fe8027f3aee&title=)

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153349416-73812439-0f41-444f-90c3-14eb03f78c36.jpeg#averageHue=%23efefef&clientId=uae7a86c3-043f-4&from=ui&id=u10120934&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=182417&status=done&style=none&taskId=u424195e8-aae9-43b9-8ef8-808cb18138c&title=)

# Locks vs. Latches

We need to make a distinction between locks and latches when discussing how the DBMS protects its internal elements.
**Locks: **A lock is a higher-level, logical primitive that protects the contents of a database (e.g., tuples, tables, databases) from other transactions. Transactions will hold a lock for its entire duration. Database systems can expose to the user which locks are being held as queries are run. Locks need to be able to rollback changes.
**Latches:**  A latch is a low-level protection primitive that the DBMS uses for the critical sections in its internal data structures (e.g., hash tables, regions of memory). Latches are held for only the duration of the operation being made. Latches do not need to be able to rollback changes.

# Buffer Pool

The *buffer pool* is an in-memory cache of pages read from disk. It is essentially a large memory region allocated inside of the database to store pages that are fetched from disk.
The buffer pool’s region of memory organized as an array of **fixed size pages.**  Each array entry is called a **frame**. When the DBMS requests a page, an exact copy is placed into one of the frames of the buffer pool. Then, the database system can search the buffer pool first when a page is requested. If the page is not found, then the system fetches a copy of the page from the disk. Dirty pages are buffered and not written back immediately（write-back cache）. See Figure 2 for a diagram of the buffer pool’s memory organization.

![Figure 2: Buffer pool organization and meta-data](https://cdn.nlark.com/yuque/0/2023/png/22382307/1678157490944-742a9b87-7da0-4a89-98fc-0f2f3aecc1de.png#averageHue=%23fbf4f3&clientId=uae7a86c3-043f-4&from=paste&height=480&id=u107c7af1&originHeight=960&originWidth=1110&originalType=binary&ratio=2&rotation=0&showTitle=true&size=101095&status=done&style=none&taskId=ud7df79f3-f453-4dac-b69b-a25de6a1457&title=Figure%202%3A%20Buffer%20pool%20organization%20and%20meta-data&width=555)

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153349892-778ced66-d7d6-4c13-a231-72e2bc312552.jpeg#averageHue=%23e9e9e9&clientId=uae7a86c3-043f-4&from=ui&id=u79f509b8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=402976&status=done&style=none&taskId=ufb153919-a47a-477e-adad-c2a3be1405f&title=)

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153350320-101aa9ff-3973-4339-9fb6-d66d3f99ee83.jpeg#averageHue=%23e9e9e9&clientId=uae7a86c3-043f-4&from=ui&id=uafe2eb9e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=396437&status=done&style=none&taskId=ubbd4529d-3b78-4373-90ab-af039c6a13b&title=)

## Buffer Pool Meta-data

The buffer pool must maintain certain meta-data in order to be used efficiently and correctly.
Firstly, the *page table* is an in-memory hash table that keeps track of pages that are currently in memory. It maps page ids to frame locations in the buffer pool. Since the order of pages in the buffer pool does not necessarily reflect the order on the disk, this extra indirection layer allows for the identification of page locations in the pool.
**Note: **The **page table** is not to be confused with the **page directory**, which is the mapping from page ids to page locations in database files. All changes to the page directory must be recorded on disk to allow the DBMS to find on restart.
The page table also maintains additional meta-data per page, a dirty-flag and a pin/reference counter.

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153350463-317bf974-8dbf-4f19-9c65-3f7e5e3ffe10.jpeg#averageHue=%23eaeaea&clientId=uae7a86c3-043f-4&from=ui&id=uf7b141fb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=356624&status=done&style=none&taskId=u3649cc3b-1f9a-4393-9925-b9273a093dc&title=)

The *dirty-flag* is set by a thread whenever it modifies a page. This indicates to storage manager that the page must be written back to disk.
The *pin/reference* Counter tracks the number of threads that are currently accessing that page (either reading or modifying it). A thread has to increment the counter before they access the page. If a page’s count is greater than zero, then the storage manager is **not** allowed to evict that page from memory.

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153351043-762701db-a635-4bf1-9a0d-d3037dcfa0e0.jpeg#averageHue=%23eaeaea&clientId=uae7a86c3-043f-4&from=ui&id=u8de2b05a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=358454&status=done&style=none&taskId=u8d98ff3f-82fd-449a-8fea-c31ced474fe&title=)

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153351606-2ef93a63-9973-4297-84e1-b3a71bb69576.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u7b04b5ab&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=348914&status=done&style=none&taskId=u65c527e7-31db-4d88-a523-8a0d2ed89aa&title=)

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153352400-c5911648-e775-4348-b72c-9cf0159dc982.jpeg#averageHue=%23f3f3f3&clientId=uae7a86c3-043f-4&from=ui&id=uc651bb8b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=537655&status=done&style=none&taskId=u46a09b36-17e0-4ebc-9640-d16f73f34e4&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153352763-ac24cc40-bb62-4b3e-8144-2ba3915217e5.jpeg#averageHue=%23eaeaea&clientId=uae7a86c3-043f-4&from=ui&id=u12fdae9b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=354869&status=done&style=none&taskId=u6cd080c7-5ce9-43f0-87cc-4777741fb15&title=)

## Memory Allocation Policies

Memory in the database is allocated for the buffer pool according to two policies.
*Global policies* deal with decisions that the DBMS should make to benefit the entire workload that is being executed. It considers all active transactions to find an optimal decision for allocating memory.
An alternative is *local policies*, which makes decisions that will make a single query or transaction run faster, even if it isn’t good for the entire workload. Local policies allocate frames to a specific transactions without considering the behavior of concurrent transactions.
Most systems use a combination of both global and local views.

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153352564-53946bc9-b383-451e-9fd6-4673dc9cb7ff.jpeg#averageHue=%23ededed&clientId=uae7a86c3-043f-4&from=ui&id=ubc9143d1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=256832&status=done&style=none&taskId=u1f07fc94-d49d-4729-9a7a-c7f06249e1b&title=)

# Buffer Pool Optimizations

There are a number of ways to optimize a buffer pool to tailor it to the application’s workload.

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153353126-1d179913-7b01-449b-8c47-3ea6b1db78f0.jpeg#averageHue=%23efefef&clientId=uae7a86c3-043f-4&from=ui&id=u65afd2eb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=194590&status=done&style=none&taskId=uafc796da-b53a-4ef9-ac06-2a4a3e0cf00&title=)

## Multiple Buffer Pools

The DBMS can maintain multiple buffer pools for different purposes (i.e per-database buffer pool, per-page type buffer pool). Then, each buffer pool can adopt local policies tailored for the data stored inside of it. This method can help reduce latch contention and improves locality.
Two approaches to mapping desired pages to a buffer pool are object IDs and hashing.
_Object IDs _involve extending the record IDs to have an object identifier. Then through the object identifier, a mapping from objects to specific buffer pools can be maintained.
Another approach is *hashing* where the DBMS hashes the page id to select which buffer pool to access.

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153353595-ec002489-d228-4595-89f8-21c00752c522.jpeg#averageHue=%23eaeaea&clientId=uae7a86c3-043f-4&from=ui&id=uc0b8f752&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=372983&status=done&style=none&taskId=ua4f9a9a1-f02c-4060-aed0-f80cee27402&title=)

bufferpool -> tablespace -> table

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153354566-eb795e51-4623-4624-93ac-0d40380784dc.jpeg#averageHue=%23e7e7e7&clientId=uae7a86c3-043f-4&from=ui&id=uf0944fa3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=276097&status=done&style=none&taskId=u244271b4-3fb2-49cb-b4da-ae354bbf4ce&title=)

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153355297-d7e24410-688d-4e10-9f87-ca03b40203ba.jpeg#averageHue=%23ebeaea&clientId=uae7a86c3-043f-4&from=ui&id=u13b80b12&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=418252&status=done&style=none&taskId=u12e9e96c-e00a-4f45-824a-21fad67c48f&title=)

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153355297-0567d3a4-9794-42fb-96bf-de6365c3d931.jpeg#averageHue=%23ebeaea&clientId=uae7a86c3-043f-4&from=ui&id=u23d5f88a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=407105&status=done&style=none&taskId=u10af2c94-ec1f-414a-aca6-c54114a4438&title=)

## Pre-fetching

The DBMS can also optimize by pre-fetching pages based on the query plan. Then, while the first set of pages is being processed, the second can be pre-fetched into the buffer pool. This method is commonly used by DBMS’s when accessing many pages sequentially.

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153355238-8ee7a04f-cb67-4f70-a7cc-a43b9f28dd0a.jpeg#averageHue=%23eae9e9&clientId=uae7a86c3-043f-4&from=ui&id=u2834bb7a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=329012&status=done&style=none&taskId=u2615e3e3-326c-469a-9c18-f3484b9280a&title=)

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153355717-af80b53f-35c5-4b52-93c1-bbe89f56b696.jpeg#averageHue=%23e9e9e9&clientId=uae7a86c3-043f-4&from=ui&id=u32d57d3e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=328489&status=done&style=none&taskId=u0febe001-75d0-42e9-9e59-83291decb96&title=)

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153356804-9af370fe-bd9c-4449-9c4d-115b9e2ebf57.jpeg#averageHue=%23e8e8e8&clientId=uae7a86c3-043f-4&from=ui&id=u7cfc772f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=343250&status=done&style=none&taskId=u71896757-389f-4ebe-8399-4366d37bb73&title=)

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153357138-773eb9ee-b908-4b67-a404-c969bcfcdf7c.jpeg#averageHue=%23e8e8e8&clientId=uae7a86c3-043f-4&from=ui&id=u62170299&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=317113&status=done&style=none&taskId=u4ad9aeb8-38b9-4de4-8d42-f8551546d66&title=)

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153357336-522c5bd9-d26c-42e8-8d6b-e2b4d758bab4.jpeg#averageHue=%23e8e8e8&clientId=uae7a86c3-043f-4&from=ui&id=uaea6c0bd&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=317390&status=done&style=none&taskId=ubdaece3d-50eb-4c8d-9378-c237c2f0a49&title=)

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153357355-8e9e2118-27b2-49a9-972f-cbafaf3765d7.jpeg#averageHue=%23e7e7e7&clientId=uae7a86c3-043f-4&from=ui&id=uabc5944f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=315628&status=done&style=none&taskId=u44c0a7a8-98d5-44a8-9feb-92282551c18&title=)

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153357676-ede328bb-a3ba-4bba-b332-8b89166ddc77.jpeg#averageHue=%23e7e7e7&clientId=uae7a86c3-043f-4&from=ui&id=uedc4c14a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=365038&status=done&style=none&taskId=u01729a5d-3f48-40da-902c-3b36c587450&title=)

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153359034-12bc18f5-3768-4ec1-8ef1-9a12ce4679bd.jpeg#averageHue=%23e7e6e6&clientId=uae7a86c3-043f-4&from=ui&id=ubd97e34d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=367850&status=done&style=none&taskId=uabf48e92-3825-4b25-adc3-3c9c5932d7a&title=)

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153359481-6c34b9b6-f8d0-4723-9dbb-16d38526aae1.jpeg#averageHue=%23e7e6e6&clientId=uae7a86c3-043f-4&from=ui&id=uf4960283&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=382791&status=done&style=none&taskId=ua042f8fc-5f10-47ca-9409-3c24e0a8063&title=)

## Scan Sharing (Synchronized Scans)

Query cursors can reuse data retrieved from storage or operator computations. This allows multiple queries to attach to a single cursor that scans a table. Ifb a query starts a scan and if there one already doing this, then the DBMS will attach the second query’s cursor to the existing cursor. The DBMS keeps track of where the second query joined with the first so that it can finish the scan when it reaches the end of the data structure.

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153359291-97da5221-c917-4c04-98d7-3ca13bccc931.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u80aadf73&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=309248&status=done&style=none&taskId=u37656fa0-3d4f-4704-a903-0d184fec8ee&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153359453-6ea90766-f180-497e-a793-ade446fc481c.jpeg#averageHue=%23ebeaea&clientId=uae7a86c3-043f-4&from=ui&id=uffb08acb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=364127&status=done&style=none&taskId=ube2e39ae-ddd6-4cdf-9a90-10047633559&title=)

Problems that Synchronized Scans might incur:

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153360184-39378148-2528-4212-a6d7-534d9c8ffe12.jpeg#averageHue=%23f6f6f6&clientId=uae7a86c3-043f-4&from=ui&id=u1fa28da9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=473297&status=done&style=none&taskId=u5325dd29-dd74-4383-87fb-479cfa46c56&title=)

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153361530-cc02527a-ae9b-4867-9452-790a11e441fb.jpeg#averageHue=%23f5f5f4&clientId=uae7a86c3-043f-4&from=ui&id=u81c8ae48&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=551260&status=done&style=none&taskId=u884d8cf3-45da-473e-874f-7c1364ae2a0&title=)

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153361130-e5b0a992-4db5-4528-9df3-b1d6cd9a3105.jpeg#averageHue=%23e9e9e9&clientId=uae7a86c3-043f-4&from=ui&id=u60aa2c60&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=293405&status=done&style=none&taskId=u70c778f1-7b14-4a87-8209-7f292f80075&title=)

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153361335-5f43dc4f-10da-47f5-bef2-7d0ac3c8cd55.jpeg#averageHue=%23e7e7e7&clientId=uae7a86c3-043f-4&from=ui&id=u4c25223b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=290356&status=done&style=none&taskId=u56d57793-7d2a-4b18-9072-a6bc931d6b4&title=)

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153361563-08a742c6-25d6-4691-9ab2-c509817c886e.jpeg#averageHue=%23e8e7e7&clientId=uae7a86c3-043f-4&from=ui&id=ue1d86486&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=294185&status=done&style=none&taskId=u1ce112a1-0f87-4457-bfdb-155733672e5&title=)

![37.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153362073-e93a65d1-41a5-468c-9f2c-ed812694dae7.jpeg#averageHue=%23e8e7e7&clientId=uae7a86c3-043f-4&from=ui&id=u4a15a5c8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=294197&status=done&style=none&taskId=u415de203-a90b-40ea-92e4-3e0c369ae7b&title=)

![38.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153363064-808cf23a-11ca-4a44-941c-8fd5a5d64813.jpeg#averageHue=%23e6e6e6&clientId=uae7a86c3-043f-4&from=ui&id=udcc70749&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=316583&status=done&style=none&taskId=uc04897ee-fbfb-4769-b293-851021e2a0d&title=)

![39.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153363587-d54260b6-49bf-4aa8-9c52-40de1c584417.jpeg#averageHue=%23e6e6e6&clientId=uae7a86c3-043f-4&from=ui&id=u9b1da93a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=320326&status=done&style=none&taskId=u16c867d1-7bce-448a-9ad1-c468af2eb21&title=)

![40.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153363599-8856ce13-a3bb-44da-91c1-c1b981dce7dc.jpeg#averageHue=%23e6e6e6&clientId=uae7a86c3-043f-4&from=ui&id=ubb198c2f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319326&status=done&style=none&taskId=u92a52e2f-5ed2-4aad-a629-0b75ffe4dff&title=)

![41.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153364058-00543f1d-7a9d-4652-b81e-c978e7b2ab27.jpeg#averageHue=%23e6e6e6&clientId=uae7a86c3-043f-4&from=ui&id=u81dc89f5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319513&status=done&style=none&taskId=ubf8867fb-193a-4770-8dc0-aa5cf16d34a&title=)

![42.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153364063-47a633ca-277b-42f9-9b35-97a23061655d.jpeg#averageHue=%23e6e6e6&clientId=uae7a86c3-043f-4&from=ui&id=u2281b217&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=316541&status=done&style=none&taskId=ucdd6aa12-7df8-4cb3-bfce-816e6284326&title=)

![43.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153365427-10d63e1b-02b4-4ded-8253-4a29d1ba3993.jpeg#averageHue=%23e6e6e6&clientId=uae7a86c3-043f-4&from=ui&id=ubc1feeca&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=316598&status=done&style=none&taskId=u6f74757e-b6fe-43bb-90ea-9c3ea674104&title=)

![44.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153365529-f6531f67-a7f0-4a59-83ef-3601d500c48c.jpeg#averageHue=%23e5e5e5&clientId=uae7a86c3-043f-4&from=ui&id=u617f5898&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=325468&status=done&style=none&taskId=ud1aea4c0-ffb1-4ae4-a2c0-a21a3a628c2&title=)

## Buffer Pool Bypass

The sequential scan operator will not store fetched pages in the buffer pool to avoid overhead. Instead, memory is local to the running query. This works well if operator needs to read a large sequence of pages that are contiguous on disk. Buffer Pool Bypass can also be used for temporary data (sorting, joins).

![45.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153365937-37718a9f-8ebf-4e5d-bca9-b11767245341.jpeg#averageHue=%23ebeaea&clientId=uae7a86c3-043f-4&from=ui&id=u36cf568a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=385407&status=done&style=none&taskId=uf9ead3d6-6ef8-4a5f-bbad-ea2382d141f&title=)

# OS Page Cache

Most disk operations go through the OS API. Unless explicitly told otherwise, the OS maintains its own filesystem cache.
Most DBMS use **direct I/O to bypass the OS’s cache** in order to avoid redundant copies of pages and having to manage different eviction policies.
**Postgres** is an example of a database system that uses the OS’s Page Cache.

![46.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153366172-21fe8fa4-79d2-4867-b753-54eb41b1b48e.jpeg#averageHue=%23e4e4e4&clientId=uae7a86c3-043f-4&from=ui&id=u2b5b0938&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=381784&status=done&style=none&taskId=u89aaf7b6-a617-40a1-8d63-7e6c7536a15&title=)

![47.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153366285-12dc6b27-a955-4ea6-bf0c-2fd1c4da8a22.jpeg#averageHue=%23e4e4e4&clientId=uae7a86c3-043f-4&from=ui&id=u183a458a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=400337&status=done&style=none&taskId=u90d88421-e509-462c-91cd-0777e64dd7c&title=)

![48.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153367333-85e31de8-2df6-4dcc-936e-b973d4badd0b.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u1d83e579&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=324362&status=done&style=none&taskId=u2215e9d8-72a4-46c5-b07e-17814c80686&title=)

# Buffer Replacement Policies

When the DBMS needs to free up a frame to make room for a new page, it must decide which page to evict from the buffer pool.
A replacement policy is an algorithm that the DBMS implements that makes a decision on which pages to evict from buffer pool when it needs space.
Implementation goals of replacement policies are improved correctness, accuracy, speed, and meta-data overhead.

![49.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153367319-d6da88b3-a735-4cac-97b0-5cd1f3b93cce.jpeg#averageHue=%23ededed&clientId=uae7a86c3-043f-4&from=ui&id=u752f0499&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=267108&status=done&style=none&taskId=ua24c5492-4e6a-4bb4-b790-5c459ffc584&title=)

## Least Recently Used (LRU)

The Least Recently Used replacement policy maintains a timestamp of when each page was last accessed. The DBMS picks to evict the page with the oldest timestamp. This timestamp can be stored in a separate data structure, such as a queue, to allow for sorting and improve efficiency by reducing sort time on eviction.

![50.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153367993-ea1e1b02-6d91-445f-81f6-c6c80a8e79e5.jpeg#averageHue=%23ededed&clientId=uae7a86c3-043f-4&from=ui&id=uf4dfa381&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=275949&status=done&style=none&taskId=ua322d8a4-bd10-4587-b0c5-d3860849fa7&title=)

## CLOCK

The CLOCK policy is an approximation of LRU without needing a separate timestamp per page. In the CLOCK policy, each page is given a **reference bit**. When a page is accessed, set to 1.
To visualize this, organize the pages in a circular buffer with a “clock hand”. Upon sweeping check if a page’s bit is set to 1. If yes, set to zero, if no, then evict it. In this way, the clock hand remembers position between evictions.

![Figure 3: Visualization of CLOCK replacement policy. Page 1 is referenced and set to 1. When the clock hand sweeps, it sets the reference bit for page 1 to 0 and evicts page 5.](https://cdn.nlark.com/yuque/0/2023/png/22382307/1678174395868-e0c97f8b-d212-4c55-b63a-9b322fc3f6e4.png#averageHue=%23fcf4f3&clientId=uae7a86c3-043f-4&from=paste&height=394&id=u6deac127&originHeight=788&originWidth=1556&originalType=binary&ratio=2&rotation=0&showTitle=true&size=66465&status=done&style=none&taskId=u06432995-b0a0-408e-8729-2071ea017e7&title=Figure%203%3A%20Visualization%20of%20CLOCK%20replacement%20policy.%20Page%201%20is%20referenced%20and%20set%20to%201.%20When%20the%20clock%20hand%20sweeps%2C%20it%20sets%20the%20reference%20bit%20for%20page%201%20to%200%20and%20evicts%20page%205.&width=778)

![51.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153368468-f72a04a3-8710-4884-9de1-efe86bb28118.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=ub8098847&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=352413&status=done&style=none&taskId=uda13dca6-4386-4d05-9801-8cc0d6193c9&title=)

![52.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153368635-1d42ab91-85ab-4412-bb26-3e76dd4f0f3b.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=ue4ebe328&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=351302&status=done&style=none&taskId=u5f56b590-980e-4985-8901-e5ac13ea461&title=)

![53.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153369195-35cc5ba2-ff5f-43ba-ad89-15489b88f494.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u2974afa2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=346826&status=done&style=none&taskId=u8a361dc9-d570-499a-9434-0f7c9260393&title=)

![54.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153369471-11590e4b-677a-4507-ae94-22e155ba9b3f.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u774398f2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=346791&status=done&style=none&taskId=u039ba86c-e934-4378-9db4-9f379c607ed&title=)

![55.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153370533-65e8af5a-74fa-4871-946d-74bb08a8850e.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=uba2d2229&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=346512&status=done&style=none&taskId=u2e7509b1-7adf-40e2-a34b-ec2d7a21a69&title=)

![56.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153370803-1bee71cf-bc9f-448c-9605-de60081425a8.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u5d8ae11a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=344039&status=done&style=none&taskId=ue1cc4b5b-134a-498e-b2bb-5ed6fed5145&title=)

![57.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153370790-7b94cbe7-e8c9-44d9-a3bf-9348ea37c3a8.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u3244b8c4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=346841&status=done&style=none&taskId=u3f177dec-1c82-4caa-81c7-740fca927c9&title=)

![58.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153371304-3de34c04-dffd-418f-a229-e975e2ab13ef.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u4edb0bd6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=346841&status=done&style=none&taskId=ud20d5a52-289e-4b1e-aba7-45c7f1d91e6&title=)

![59.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153371404-6125964d-c5cd-4867-8445-dcc6db05b0f2.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=u9965fc61&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=313573&status=done&style=none&taskId=ue7636ef6-cab4-4041-83c3-bc42c700489&title=)

![60.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153372689-abd86d66-e99e-45c8-9099-8fc25bdc323c.jpeg#averageHue=%23e7e7e7&clientId=uae7a86c3-043f-4&from=ui&id=u4c3fdd54&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=301040&status=done&style=none&taskId=u46d97cad-4a0b-49c0-ba95-c92f64a3d20&title=)

![61.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153372919-6ae9f3d8-85dd-4fa3-9ca3-33750266b766.jpeg#averageHue=%23e5e5e5&clientId=uae7a86c3-043f-4&from=ui&id=u27c3b822&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=327419&status=done&style=none&taskId=u096840bb-1c0c-489d-9919-07c64123c03&title=)

![62.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153373013-3e0652db-fcb4-4771-90dd-2054e413f0d6.jpeg#averageHue=%23e4e4e4&clientId=uae7a86c3-043f-4&from=ui&id=uaff16852&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=325359&status=done&style=none&taskId=uea6db57d-bc58-49be-ba9e-3aa09a5c521&title=)

![63.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153373455-156aec37-003f-4178-8e6d-c9204d10d6c9.jpeg#averageHue=%23e4e4e4&clientId=uae7a86c3-043f-4&from=ui&id=u09d61dad&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=329202&status=done&style=none&taskId=uc2ebd799-19ae-4382-961d-9e5fe9787ae&title=)

![64.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153373560-eeddeb65-4104-47ec-9a46-3c6c5ef91fbf.jpeg#averageHue=%23e4e4e4&clientId=uae7a86c3-043f-4&from=ui&id=u26f8f2a3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=325314&status=done&style=none&taskId=u212fe48c-14c5-4a47-9ce3-1d0cbada742&title=)

![65.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153374750-e1cfdc0d-352c-43c5-8396-6daa81d0b40a.jpeg#averageHue=%23e2e2e2&clientId=uae7a86c3-043f-4&from=ui&id=u670f0190&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=364304&status=done&style=none&taskId=uc833b2fc-3838-415e-a0b7-53d820b5137&title=)

## Alternatives

There are a number of problems with LRU and CLOCK replacement policies.
Namely, LRU and CLOCK are susceptible to *sequential flooding*, where the buffer pool’s contents are corrupted due to a sequential scan. Since sequential scans read every page, the timestamps of pages read may not reflect which pages we actually want. In other words, the most recently used page is actually the most unneeded page.
There are three solutions to address the shortcomings of LRU and CLOCK policies.
One solution is **LRU-K **which tracks the history of the last K references as timestamps and computes the interval between subsequent accesses. This history is used to predict the next time a page is going to be accessed.

![66.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153374904-d60efdd3-0a76-4670-80a2-f8489f78535d.jpeg#averageHue=%23ededed&clientId=uae7a86c3-043f-4&from=ui&id=u786a6817&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=285850&status=done&style=none&taskId=u14461d97-4d24-4225-9cc9-6e58aad319d&title=)

Another optimization is localization per query. The DBMS chooses which pages to evict on a per transaction/query basis. This minimizes the pollution of the buffer pool from each query.

![67.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153374840-4491b5e2-794f-43c0-8a51-b5bcf4ad65cf.jpeg#averageHue=%23ebebeb&clientId=uae7a86c3-043f-4&from=ui&id=u8588591c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=314654&status=done&style=none&taskId=ue1de9632-5bab-4e78-90cb-f4ec43e6b5c&title=)

Lastly, priority hints allow transactions to tell the buffer pool whether page is important or not based on the context of each page during query execution.

![68.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153375485-b8bb1a35-ea92-4cf5-bf7b-0c249a429ef5.jpeg#averageHue=%23eae9e9&clientId=uae7a86c3-043f-4&from=ui&id=uf89ce0a5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=357572&status=done&style=none&taskId=u7ad68b27-e568-47bc-80b6-a6bbc35ecad&title=)

![69.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153375609-802cfdfb-80d5-4e5a-86cc-e400e58414d2.jpeg#averageHue=%23e7e7e7&clientId=uae7a86c3-043f-4&from=ui&id=u3ebf8997&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=383240&status=done&style=none&taskId=u9cf4ce13-502f-488b-b383-c168776f4c8&title=)

## Dirty Pages

There are two methods to handling pages with dirty bits. The fastest option is to drop any page in the buffer pool that is not dirty. A slower method is to write back dirty pages to disk to ensure that its changes are persisted.
These two methods illustrate the trade-off between fast evictions versus dirty writing pages that will not be read again in the future.

![70.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153377123-995eee76-db12-4c9d-a031-89199343e936.jpeg#averageHue=%23ebebeb&clientId=uae7a86c3-043f-4&from=ui&id=u42103909&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=333088&status=done&style=none&taskId=ud2e0f970-17b6-474b-959d-8db0075f821&title=)

One way to avoid the problem of having to write out pages unnecessarily is background writing. Through background writing, the DBMS can periodically walk through the page table and write dirty pages to disk. When a dirty page is safely written, the DBMS can either evict the page or just unset the dirty flag.

![71.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153377211-404b5116-5d2f-4f80-96a7-9e6699791968.jpeg#averageHue=%23ebebeb&clientId=uae7a86c3-043f-4&from=ui&id=u88cec0ad&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=336364&status=done&style=none&taskId=uaeae6c18-6121-4450-8f73-60defd443aa&title=)

# Other Memory Pools

The DBMS needs memory for things other than just tuples and indexes. These other memory pools may not always backed by disk depending on implementation.

- Sorting + Join Buffers
- Query Caches
- Maintenance Buffers
- Log Buffers
- Dictionary Caches

![72.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153377213-e068ce30-83b5-4ff3-b23a-6f99fe9e68db.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=ub6b4d5e5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=304150&status=done&style=none&taskId=u8073a766-59f9-4b98-b78b-30fcd4d021d&title=)

![73.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153377511-24127ce2-d558-4097-b250-bb17f6e31f9c.jpeg#averageHue=%23eeeeee&clientId=uae7a86c3-043f-4&from=ui&id=uee403502&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=245951&status=done&style=none&taskId=uffacaafb-699e-4d95-a0c9-ee8c50c60b0&title=)

![74.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153377406-e444126d-c91d-41cb-af4b-637f317aa960.jpeg#averageHue=%23f1f1f1&clientId=uae7a86c3-043f-4&from=ui&id=u5a4eaaa1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=139113&status=done&style=none&taskId=u99e00139-4aef-4f22-be05-4828403bd4d&title=)

![75.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153379374-a3b3da12-cce1-4ebf-ae13-a50cc33c2a9a.jpeg#averageHue=%23e9e9e9&clientId=uae7a86c3-043f-4&from=ui&id=ucc005a24&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=402613&status=done&style=none&taskId=uea2a5ed7-66b6-44e3-b11c-c227060be13&title=)

![76.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153379272-0072a04e-713e-4581-8ef0-26500494c01a.jpeg#averageHue=%23ebeaea&clientId=uae7a86c3-043f-4&from=ui&id=u0fe48705&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=371130&status=done&style=none&taskId=u06d6e492-c908-424f-a380-c188b483637&title=)

![77.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153379357-a9132a1f-5bdd-481b-a00b-029498dcb7f8.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=uafba2031&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=296394&status=done&style=none&taskId=u185225ae-23c0-4b39-b9db-9e0008d05a7&title=)

![78.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153379604-77e68d5f-5885-42ae-ac37-0e0da4809c22.jpeg#averageHue=%23e7e7e7&clientId=uae7a86c3-043f-4&from=ui&id=u57c9babd&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=439050&status=done&style=none&taskId=u84555ff3-71af-4197-98a1-b9211510679&title=)

![79.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153379376-c4afc67a-0f69-48d0-8792-236224107d41.jpeg#averageHue=%23ececec&clientId=uae7a86c3-043f-4&from=ui&id=ud27c4364&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=299807&status=done&style=none&taskId=u5c906a68-b588-41ea-8644-b03e4c494da&title=)

![80.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153381589-0e69fc7e-127a-4f8e-9560-3e031ef0c8c4.jpeg#averageHue=%23eeeded&clientId=uae7a86c3-043f-4&from=ui&id=ud1f3c012&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319528&status=done&style=none&taskId=u55be341a-2521-42f7-8e28-28187cf9eeb&title=)

![81.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153381728-157fa72a-575a-4bc3-8b06-2362d154a09d.jpeg#averageHue=%23ebebeb&clientId=uae7a86c3-043f-4&from=ui&id=u6ffd16e7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=356675&status=done&style=none&taskId=u6156a315-d3fb-4a69-a876-52c669d49ef&title=)

![82.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678153381851-571cd62f-8879-423f-bd80-929b95f3364b.jpeg#averageHue=%23ebebeb&clientId=uae7a86c3-043f-4&from=ui&id=uc70e8c7d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=383894&status=done&style=none&taskId=u620946da-a55b-4c3e-b66b-c3fb28dfef0&title=)
