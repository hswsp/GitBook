# 07 - Hash Tables

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194215679-c204f2bb-9bf2-4c79-bc68-638a2a839b6d.jpeg#averageHue=%231c2534&clientId=u10bf73b9-9427-4&from=ui&id=ubb3ffa6e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=580572&status=done&style=none&taskId=uf9fdea16-6231-4b75-8dc3-149ec139d37&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194208917-d4c3ec15-3f4b-4132-b7d2-f54784b9eba4.jpeg#averageHue=%23eeeeee&clientId=u10bf73b9-9427-4&from=ui&id=u3c0b89c6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=264075&status=done&style=none&taskId=uc166f18b-0a9a-4b2c-a7e7-2a2611a97c0&title=)

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194210453-3f602615-7d30-466d-9ef4-6bc58addf50e.jpeg#averageHue=%23eeeeee&clientId=u10bf73b9-9427-4&from=ui&id=u351307a6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=312665&status=done&style=none&taskId=u12a3cdf0-2a47-4eed-a8ff-6654cbcdcf3&title=)

# Data Structures

A DBMS uses various data structures for many different parts of the system internals. Some examples include:

- **Internal Meta-Data**: This is data that keeps track of information about the database and the system state. Ex: Page tables, page directories
- **Core Data Storage:**  Data structures are used as the base storage for tuples in the database.
- **Temporary Data Structures**: The DBMS can build data structures on the fly while processing a query to speed up execution (e.g., hash tables for joins).
- **Table Indices:**  Auxiliary data structures can be used to make it easier to find specific tuples.

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194210947-d0186892-3118-42c3-88ac-826aff39f7f2.jpeg#averageHue=%23dddddd&clientId=u10bf73b9-9427-4&from=ui&id=u68c9a0e7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=343807&status=done&style=none&taskId=uae65fe1d-35ad-4808-b5a8-c3ded46d067&title=)

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194207241-4e84a3dc-16d1-40db-87bc-624626719c02.jpeg#averageHue=%23efefef&clientId=u10bf73b9-9427-4&from=ui&id=ud7d6ee52&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=190740&status=done&style=none&taskId=ubf9a946b-5ee5-41a0-b021-1ade1f46170&title=)

There are two main design decisions to consider when implementing data structures for the DBMS:

1. **Data organization**: We need to figure out how to layout memory and what information to store inside the data structure in order to support efficient access.
2. **Concurrency**: We also need to think about how to enable multiple threads to access the data structure without causing problems.

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194214817-96f8cabb-4c1d-45a2-ae0a-60c0b97f2a9f.jpeg#averageHue=%23ededed&clientId=u10bf73b9-9427-4&from=ui&id=u450a2bd9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=276351&status=done&style=none&taskId=ud47f72ea-ed9b-4b7f-9fbd-c054a25e3b2&title=)

面对海量数据，如何快速的找到我们想要的数据？为了解决这个问题，数据库中引入了一个新的数据结构：索引。它可以使得我们通过部分attribute的值快速定位到整个tuple，让查询速度变得更快。但代价是每次写tuple的时候，都需要同时更新索引。
接下来介绍数据库中的索引的数据结构以及它们的trade-off

# Hash Table

A hash table implements an associative array abstract data type that maps keys to values. It provides on average $O (1)$operation complexity ($O (n)$in the worst-case) and $O (n)$ storage complexity. Note that even with $O (1)$operation complexity on average, there are constant factor optimizations which are important to consider in the real world.
A hash table implementation is comprised of two parts:

- **Hash Function: **This tells us how to map a large key space into a smaller domain. It is used to compute an index into an array of buckets or slots. We need to consider the trade-off between fast execution and collision rate. On one extreme, we have a hash function that always returns a constant (very fast, but everything is a collision). On the other extreme, we have a “perfect” hashing function where there are no collisions, but would take extremely long to compute. The ideal design is somewhere in the middle.
- **Hashing Scheme:**  This tells us how to handle key collisions after hashing. Here, we need to consider the trade-off between allocating a large hash table to reduce collisions and having to execute additional instructions when a collision occurs.

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194218152-67536cc2-7d63-48b2-a378-92e14ee1fba3.jpeg#averageHue=%23ececec&clientId=u10bf73b9-9427-4&from=ui&id=u66ec063f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=356745&status=done&style=none&taskId=u8c468136-26cd-427b-a7b5-852945f2316&title=)

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194219431-66897b46-fc8c-445c-a59b-cbdd3a1566e2.jpeg#averageHue=%23ededed&clientId=u10bf73b9-9427-4&from=ui&id=u005fe0ce&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=273604&status=done&style=none&taskId=ua4913282-39df-4a2a-aa55-e7994c72475&title=)

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194220630-95f30018-eaf5-442a-8e7a-14169d7ae33e.jpeg#averageHue=%23ededed&clientId=u10bf73b9-9427-4&from=ui&id=ub93d6b94&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=308507&status=done&style=none&taskId=u83435d7b-80fc-4bff-9707-7c194f3dd97&title=)

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194227610-73e82928-3429-4efd-bb68-45f9f54c58ea.jpeg#averageHue=%23ecebeb&clientId=u10bf73b9-9427-4&from=ui&id=udcb6ffb2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=342377&status=done&style=none&taskId=ue1fe4df6-879b-4422-afcc-3073118a381&title=)

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194227527-9bc88667-25e2-470c-8ccf-dd0963446466.jpeg#averageHue=%23ececec&clientId=u10bf73b9-9427-4&from=ui&id=u76121cbf&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=316566&status=done&style=none&taskId=u8cf02f7a-77a5-4904-8690-152fa4cd35d&title=)

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194226317-653d3b55-2904-4c2e-866f-1d6b0ad22048.jpeg#averageHue=%23efefef&clientId=u10bf73b9-9427-4&from=ui&id=u1f6fb3db&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=184786&status=done&style=none&taskId=u2cdbc13f-2233-4fa8-aebf-81354cb5139&title=)

# Hash Functions

A *hash function* takes in any key as its input. It then returns an integer representation of that key (i.e., the “hash”). The function’s output is deterministic (i.e., the same key should always generate the same hash output).
The DBMS need not use a cryptographically secure hash function (e.g., SHA-256) because we do not need to worry about protecting the contents of keys. These hash functions are primarily used internally by the DBMS and thus information is not leaked outside of the system. In general, we only care about the hash function’s speed and collision rate.
The current state-of-the-art hash function is Facebook XXHash3.

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194229278-3d555330-1828-4ec3-a3e1-4100bbaa1298.jpeg#averageHue=%23ededed&clientId=u10bf73b9-9427-4&from=ui&id=u1025b74d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=279701&status=done&style=none&taskId=ubc20594f-ff10-4764-a705-b41d5a0730a&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194233167-7846712c-ec59-4af0-9ff1-6ce1b478d4f2.jpeg#averageHue=%23edebeb&clientId=u10bf73b9-9427-4&from=ui&id=u028f3d4f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=412623&status=done&style=none&taskId=u5078ef7c-1376-4604-9a4a-eb529c79fc1&title=)

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194236962-041318ab-cfc9-4760-be4c-2366d0032ffd.jpeg#averageHue=%23ebebeb&clientId=u10bf73b9-9427-4&from=ui&id=u9c84a94f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=448271&status=done&style=none&taskId=u16b0a885-aa92-4991-a16a-6a38868c638&title=)

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194240290-d50e8447-2897-48b7-b713-441347f00795.jpeg#averageHue=%23ecede6&clientId=u10bf73b9-9427-4&from=ui&id=u6bf78a23&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=461312&status=done&style=none&taskId=u41c16de9-8312-4a1a-82c5-2af97ba5c1b&title=)

# Static Hashing Schemes

A static hashing scheme is one where the size of the hash table is fixed. This means that if the DBMS runs out of storage space in the hash table, then it has to rebuild a larger hash table from scratch, which is very expensive. Typically the new hash table is twice the size of the original hash table.
To reduce the number of wasteful comparisons, it is important to avoid collisions of hashed key. Typically, we use twice the number of slots as the number of expected elements.
The following assumptions usually do not hold in reality:

1. The number of elements is known ahead of time.
2. Keys are unique.
3. There exists a perfect hash function.

Therefore, we need to choose the hash function and hashing schema appropriately.

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194234829-4d5e55b7-0a59-4a97-a63f-d88678bfa1f7.jpeg#averageHue=%23ededed&clientId=u10bf73b9-9427-4&from=ui&id=u2f3c040c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=222256&status=done&style=none&taskId=ub024df87-61b8-4da5-b3b7-3c95dc48cb2&title=)

## Linear Probe Hashing

This is the most basic hashing scheme. It is also typically the fastest. It uses a circular buffer of array slots. The hash function maps keys to slots. When a collision occurs, we linearly seach the adjacent slots until an open one is found. For lookups, we can check the slot the key hashes to, and search linearly until we find the desired entry. If we reach an empty slot or we iterated over every slot in the hashtable, then the key is not in the table. Note that this means we have to store both key and value in the slot so that we can check if an entry is the desired one. Deletions are more tricky. **We have to be careful about just removing the entry from the slot, as this may prevent future lookups from finding entries that have been put below the now empty slot.（注：因为这里我们查找一个数是否存在的标准就是找到第一个empty slot就认为不存在了。）**  There are two solutions to this problem:

- The most common approach is to use “tombstones”. Instead of deleting the entry, we replace it with a “tombstone” entry which tells future lookups to keep scanning.
- The other option is to shift the adjacent data after deleting an entry to fill the now empty slot. However, we must be careful to **only move the entries which were originally shifted.（不能移动**​$hash(key)$​**就在当前位置的数）**  This is rarely implemented in practice.

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194238474-f6b8d5a0-91ca-4ab7-af98-9512915ebe8f.jpeg#averageHue=%23ececec&clientId=u10bf73b9-9427-4&from=ui&id=u9c261fbf&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=322550&status=done&style=none&taskId=uc9b06bb0-17d7-43f2-82be-38570f04c21&title=)

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194240287-5235c42e-9e34-412c-9f51-605d3d8ed690.jpeg#averageHue=%23edecec&clientId=u10bf73b9-9427-4&from=ui&id=u48d9ed8a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=252576&status=done&style=none&taskId=u449f4766-24c5-4d05-aada-ecff7162831&title=)

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194241502-7fc4143d-4afc-4fe3-9d12-aff15fd3c99f.jpeg#averageHue=%23eeeded&clientId=u10bf73b9-9427-4&from=ui&id=u02f062b1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=225541&status=done&style=none&taskId=u4aca548e-519c-445a-867e-7ff9ef0de38&title=)

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194243037-d261cbc1-0aef-4e68-8392-e271cc042b70.jpeg#averageHue=%23eeeded&clientId=u10bf73b9-9427-4&from=ui&id=u93cbe524&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=230893&status=done&style=none&taskId=uc3e6abc9-ade9-4005-8824-a639ff7480b&title=)

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194244765-98e22d0d-5750-48d0-94cf-e3e540ed1717.jpeg#averageHue=%23ededed&clientId=u10bf73b9-9427-4&from=ui&id=u5096527e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=235437&status=done&style=none&taskId=u4417777e-b344-4d64-ae26-b20b0eccbd4&title=)

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194247321-4106902c-b9ea-42ec-a7f0-e4b330336ce1.jpeg#averageHue=%23edecec&clientId=u10bf73b9-9427-4&from=ui&id=ub00d4594&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=247435&status=done&style=none&taskId=u9372dc7d-e1b9-4535-aa9e-cffce4030c1&title=)

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194248321-850e6dd8-6d2d-4108-b5b2-41c22225ad19.jpeg#averageHue=%23ededed&clientId=u10bf73b9-9427-4&from=ui&id=u2bcafba8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=244396&status=done&style=none&taskId=uc9ecd3dc-8a47-4e72-b2ed-fb559a0cdfa&title=)

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194248625-053b987e-7108-4f49-b352-842997a3a979.jpeg#averageHue=%23ececec&clientId=u10bf73b9-9427-4&from=ui&id=u74b9006e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=255811&status=done&style=none&taskId=u723b97db-54b0-4b09-9aa1-c16f78473a4&title=)

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194250338-f0f12059-392d-4328-bc43-d3a905239e57.jpeg#averageHue=%23ecebeb&clientId=u10bf73b9-9427-4&from=ui&id=u1a9d369b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=254822&status=done&style=none&taskId=uee9b2a83-5eb4-4252-8b06-6a95a631e73&title=)

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194254195-ea894be9-b86f-44e6-8d13-adf89e5a02b1.jpeg#averageHue=%23e9e9e9&clientId=u10bf73b9-9427-4&from=ui&id=ueccf0a9b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=335234&status=done&style=none&taskId=u2ec9f88e-d7d3-440b-bf99-2a65a198e52&title=)

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194256361-4607b743-16d4-4f4e-97e7-5cc7a0f7fbe3.jpeg#averageHue=%23e9e9e9&clientId=u10bf73b9-9427-4&from=ui&id=u1a93f31d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=359509&status=done&style=none&taskId=udeb6d551-1a36-4ede-a34a-24f6c61bb04&title=)

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194258020-0ff23e10-4bad-48f1-b64b-d09f5786a656.jpeg#averageHue=%23e9e9e9&clientId=u10bf73b9-9427-4&from=ui&id=u3681124d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=357751&status=done&style=none&taskId=u9c10874e-077b-4f0c-9780-2744359f066&title=)

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194258772-2ca7daf4-2eb3-40c1-95ed-18e7bf322b5d.jpeg#averageHue=%23e9e9e9&clientId=u10bf73b9-9427-4&from=ui&id=ua0141b49&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=357642&status=done&style=none&taskId=ue07ae531-a46e-4eb8-8a6a-b8c13665eb2&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194259690-9d551c8d-f3aa-4d94-b581-54278635394b.jpeg#averageHue=%23e9e9e8&clientId=u10bf73b9-9427-4&from=ui&id=u61a1be6d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=358758&status=done&style=none&taskId=u96186642-a151-4ee8-99b0-e1a6bb101db&title=)

**Non-unique Keys:**  In the case where the same key may be associated with multiple different values or tuples, there are two approaches.

- Seperate Linked List: Instead of storing the values with the keys, we store a pointer to a seperate storage area which contains a linked list of all the values.
- **Redundant Keys**: The more common approach is to simply store the same key multiple times in the table. Everything with linear probing still works even if we do this.

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194264167-7476ffed-35ec-4cc8-9cf1-0183bdacf45c.jpeg#averageHue=%23e9e9e9&clientId=u10bf73b9-9427-4&from=ui&id=ubd5613a7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=397314&status=done&style=none&taskId=u5eb5f675-92aa-4cf5-a2c7-4ca764bdffe&title=)

## Robin Hood Hashing

This is an extension of linear probe hashing that seeks to reduce the maximum distance of each key from their optimal position (i.e. the original slot they were hashed to) in the hash table. This strategy steals slots from “rich” keys and gives them to “poor” keys.
In this variant, each entry also records the “distance” they are from their optimal position. Then, on each insert, if the key being inserted would be farther away from their optimal position at the current slot than the current entry’s distance, we replace the current entry and continue trying to insert the old entry farther down the table.

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194264154-8d0243dc-3cfd-48d2-bff4-a07451e2beb4.jpeg#averageHue=%23ececec&clientId=u10bf73b9-9427-4&from=ui&id=udd609829&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=320113&status=done&style=none&taskId=u407d5720-5294-4d73-a43b-10c4db75eaf&title=)

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194266474-d688bfcc-c660-4d52-889a-4d40bc95124d.jpeg#averageHue=%23ededed&clientId=u10bf73b9-9427-4&from=ui&id=ufa8b8428&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=248521&status=done&style=none&taskId=ud91c2602-e5e0-4097-b616-da791dc2a10&title=)

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194266965-b81bc3bb-ee3e-4e3f-b8c6-bed22b63de3c.jpeg#averageHue=%23eeeded&clientId=u10bf73b9-9427-4&from=ui&id=ue6cfb527&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=232137&status=done&style=none&taskId=u4ee6af06-0356-4784-8d53-f0307cdb87f&title=)

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678194268662-94a5ee16-2023-46ae-9dfd-bfc907abd487.jpeg#averageHue=%23edecec&clientId=u10bf73b9-9427-4&from=ui&id=u9d7af980&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=249647&status=done&style=none&taskId=ucaf7815e-92d8-4238-a6bc-493e79eaaa6&title=)

![37.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202122241-cb4cf514-06c0-4562-a4e3-0078133a6ddc.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=u812ad060&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=255873&status=done&style=none&taskId=ua2fd9d12-76be-457b-bdbb-c4680689777&title=)

![38.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202123482-f77545fe-6852-4e1c-a30f-2bf0050dad7a.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=u892982bb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=279533&status=done&style=none&taskId=u99758171-fba4-4036-9b2f-c5bb0a2e95c&title=)

![39.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202123354-2e9a7ea9-2c17-4b97-b1f7-899499338a0f.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=u7b97ddd7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=280174&status=done&style=none&taskId=u9939c57e-2adf-4ce1-abdc-3d2766436b4&title=)

![40.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202123454-81476c88-ac82-45d3-8027-8822de77fe6b.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=u201ae3fb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=290971&status=done&style=none&taskId=ub6c007d3-3630-4016-b92f-1dd6ce402d9&title=)

![41.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202123032-839aff60-d9ac-4e27-8f24-9840c264d0a0.jpeg#averageHue=%23ececec&clientId=u3a44d191-03af-4&from=ui&id=u4ce2c906&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=271807&status=done&style=none&taskId=ud8f52e17-efc4-4bc0-9ce4-1c790bab116&title=)

## Cuckoo Hashing

Instead of using a single hash table, this approach maintains multiple hashtables with different hash functions. The hash functions are the same algorithm (e.g., XXHash, CityHash); they generate different hashes for the same key by using different seed values.
When we insert, we check every table and choose one that has a free slot (if multiple have one, we can compare things like load factor, or more commonly, just choose a random table). If no table has a free slot, we choose (typically a random one) and evict the old entry. We then rehash the old entry into a different table. In rare cases, we may end up in a cycle. If this happens, we can rebuild all of the hash tables with new hash function seeds (less common) or *rebuild the hash tables using larger tables (more common)* .
Cuckoo hashing guarantees $O (1)$ lookups and deletions, but insertions may be more expensive.
[https://github.com/efficient/libcuckoo](https://github.com/efficient/libcuckoo)

![42.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202140810-3d110313-d53f-4136-b567-d792d0a6ea8b.jpeg#averageHue=%23ecebeb&clientId=u3a44d191-03af-4&from=ui&id=ufb1991d0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=370024&status=done&style=none&taskId=ubd132bf5-8d65-4357-80b6-4d407ef0b38&title=)

![43.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202138748-2e9e4610-6aa8-458f-ac88-8384d351c2aa.jpeg#averageHue=%23e8e8e8&clientId=u3a44d191-03af-4&from=ui&id=udda502be&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=240671&status=done&style=none&taskId=u4da0654a-ee94-476d-a364-ba05b058c8c&title=)

![44.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202138844-d876289c-5748-464d-98e0-3361e3de3766.jpeg#averageHue=%23e8e8e8&clientId=u3a44d191-03af-4&from=ui&id=uc8105397&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=238490&status=done&style=none&taskId=u8f1533e5-bd37-43c0-8b3e-fd32988a088&title=)

![45.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202139316-5ed5c219-9893-4f4d-8a5b-ae5a48ac534c.jpeg#averageHue=%23e6e6e6&clientId=u3a44d191-03af-4&from=ui&id=u73741d9a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=261729&status=done&style=none&taskId=u3d60b9e8-57a1-4954-8d7b-c41e3aee535&title=)

![46.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202140845-f234cd4b-bc13-4970-b468-fde48b4d184d.jpeg#averageHue=%23e7e7e7&clientId=u3a44d191-03af-4&from=ui&id=u96d19ec9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=262068&status=done&style=none&taskId=u4e1d54b5-9fda-42c8-80ef-a8d5c2cbc03&title=)

![47.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202152042-c0caba50-994f-4e8d-bb27-2c01ce3d0c57.jpeg#averageHue=%23e6e6e6&clientId=u3a44d191-03af-4&from=ui&id=u2e57fc98&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=292236&status=done&style=none&taskId=u4ecfcfe3-52ab-49c0-ad55-cda4464df2c&title=)

![48.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202152914-0ebd3e41-0ec3-4a13-888c-0424573f9c81.jpeg#averageHue=%23e6e6e6&clientId=u3a44d191-03af-4&from=ui&id=u105cba5f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=284931&status=done&style=none&taskId=u6b4dc738-feba-467e-a4ea-00404e2039c&title=)

![49.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202154328-01570fcd-50d0-47b4-acb7-3f4193f21f13.jpeg#averageHue=%23e6e6e6&clientId=u3a44d191-03af-4&from=ui&id=ue2ecbe01&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=284540&status=done&style=none&taskId=ud27cc9c0-4441-44b8-8137-0bd99226cb0&title=)

![50.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202154449-98d21ec0-93b8-411f-a455-0d03842d3fd7.jpeg#averageHue=%23e6e5e5&clientId=u3a44d191-03af-4&from=ui&id=u5e78e5f6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=293836&status=done&style=none&taskId=ua9541853-37a1-4bf0-b3a3-3564e49d261&title=)

![51.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202154556-024806b5-05fa-49e9-9886-4c408beac99d.jpeg#averageHue=%23e6e5e5&clientId=u3a44d191-03af-4&from=ui&id=u2ca25b03&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=294296&status=done&style=none&taskId=u8e1d00a5-1230-459c-b8af-86b85f5589d&title=)

![52.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202166544-412f2311-2a04-4625-9ca6-77c6dee05601.jpeg#averageHue=%23e6e5e5&clientId=u3a44d191-03af-4&from=ui&id=uc34c9369&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=303354&status=done&style=none&taskId=u1165e370-dde0-452b-a086-f9556974bcf&title=)

![53.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202169805-98e8e92a-87a8-459a-91b1-4feba5b72bee.jpeg#averageHue=%23e5e5e5&clientId=u3a44d191-03af-4&from=ui&id=u6f742273&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=334316&status=done&style=none&taskId=u7567655d-2c2b-4532-a853-bf4348d9681&title=)

# Dynamic Hashing Schemes

The static hashing schemes require the DBMS to know the number of elements it wants to store. Otherwise it has to rebuild the table if it needs to grow/shrink in size.
Dynamic hashing schemes are able to **resize the hash table on demand** without needing to rebuild the entire table. The schemes perform this resizing in different ways that can either maximize reads or writes.

![54.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202169340-d2f05f27-38a5-47c4-9805-0bb912e3c5b7.jpeg#averageHue=%23ececec&clientId=u3a44d191-03af-4&from=ui&id=ubab556bf&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319666&status=done&style=none&taskId=u4cb4f77f-d27b-439e-8f6a-97062465867&title=)

## Chained Hashing

This is the most common dynamic hashing scheme. The DBMS maintains a linked list of buckets for each slot in the hash table. Keys which hash to the same slot are simply inserted into the linked list for that slot.

![55.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202168948-048c9ec9-e1e7-4bdf-8c45-93864b0c3d6f.jpeg#averageHue=%23ececec&clientId=u3a44d191-03af-4&from=ui&id=u3b38ea12&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=310164&status=done&style=none&taskId=u951b42ec-a0af-42da-b5eb-0a89328c86b&title=)

![56.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202170529-32156920-4bf6-438e-b355-666ce3b759f0.jpeg#averageHue=%23ebebeb&clientId=u3a44d191-03af-4&from=ui&id=u86ad94db&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=263260&status=done&style=none&taskId=ud0dba9b3-d9c2-437e-b1c7-81a9b11be4e&title=)

![57.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202179771-30f54721-9f8c-45bd-98b2-1e1752e53bf1.jpeg#averageHue=%23ebebeb&clientId=u3a44d191-03af-4&from=ui&id=ub2caccd1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=268936&status=done&style=none&taskId=u3df4a59d-2038-4c81-9bc2-48999d15f03&title=)

![58.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202182923-7cfbfb43-a4fb-4d2c-ab86-302026bc0bab.jpeg#averageHue=%23ebebeb&clientId=u3a44d191-03af-4&from=ui&id=u3c70c882&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=272727&status=done&style=none&taskId=ue566d6d7-0404-4ca1-91b1-ca884e038ce&title=)

![59.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202184612-216342bd-dfb4-48ba-a047-e535789c12e3.jpeg#averageHue=%23eaeaea&clientId=u3a44d191-03af-4&from=ui&id=u5c5b06b4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=287864&status=done&style=none&taskId=u5c131e3a-29d9-4b42-a8d9-a6ac4c4bf8e&title=)

![60.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202185552-7873ae0e-c6a5-4f0a-84ee-a8ad3696b435.jpeg#averageHue=%23eaeae9&clientId=u3a44d191-03af-4&from=ui&id=u1b1b7beb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=289148&status=done&style=none&taskId=ub841b2e9-8e86-4e3a-afa7-92effa07a4d&title=)

![61.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202186444-5e80d7ee-c5b3-4527-b208-74ad8790ca52.jpeg#averageHue=%23eae9e9&clientId=u3a44d191-03af-4&from=ui&id=u11ad77ef&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=297024&status=done&style=none&taskId=u44072d79-52fc-4687-8793-92fab9429d2&title=)

## Extendible Hashing

Improved variant of chained hashing that splits buckets instead of letting chains to grow forever. This approach allows multiple slot locations in the hash table to point to the same bucket chain.
The core idea behind re-balancing the hash table is to to move bucket entries on split and increase the number of bits to examine to find entries in the hash table. This means that the DBMS only has to move data within the buckets of the split chain; all other buckets are left untouched.

![62.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202192861-cd78f530-1362-41c9-9a26-e9d3a9964941.jpeg#averageHue=%23ececec&clientId=u3a44d191-03af-4&from=ui&id=u2ebeac30&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=322997&status=done&style=none&taskId=ub3552423-1998-4055-97fa-e564b138419&title=)

- The DBMS maintains a global and local depth bit counts that determine the number bits needed to find buckets in the slot array.
- When a bucket is full, the DBMS splits the bucket and reshuffle its elements. If the local depth of the split bucket is less than the global depth, then the new bucket is just added to the existing slot array. Otherwise, the DBMS doubles the size of the slot array to accommodate the new bucket and increments the global depth counter.

就是冲突的链表过长的时候，就需要进行分裂。即将bucket分裂，同时将**主hash表翻倍**，然后按照hash函数重新分配bucket。[https://www.geeksforgeeks.org/extendible-hashing-dynamic-approach-to-dbms/](https://www.geeksforgeeks.org/extendible-hashing-dynamic-approach-to-dbms/)

![63.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202196860-061f8baf-4db9-415e-ac5c-7e8f13c8ad86.jpeg#averageHue=%23ededed&clientId=u3a44d191-03af-4&from=ui&id=u0ec718e1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=265467&status=done&style=none&taskId=u9828ff30-82af-41b4-a01e-743fc8cc453&title=)

![64.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202198755-a332ddd0-c3d1-42ba-9856-cc292b696cf8.jpeg#averageHue=%23ededed&clientId=u3a44d191-03af-4&from=ui&id=u39eaf445&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=284443&status=done&style=none&taskId=u0742a8cc-83cd-460a-b86d-d52dc50f87f&title=)

![65.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202200944-f8ac57b5-44bd-458d-ac4b-691b19127b15.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=u5e418650&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=307152&status=done&style=none&taskId=udbf09592-c3d5-47c7-a2b0-8bb1f85ec80&title=)

`global 2`表示只examine前两个bits：

![66.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202203611-98b911a5-8108-454b-8885-5087fd265a29.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=ua2e0e2b4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=327740&status=done&style=none&taskId=u5cebf0be-8113-43af-9fdf-2196f04d03b&title=)

![67.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202210324-bb93dab3-4d5c-4347-b9c6-22727d08eb04.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=u1b5a8867&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=327943&status=done&style=none&taskId=u487bb221-8d54-4321-9964-11fc2c023c5&title=)

![68.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202211822-40ecbc14-3351-4100-842d-59fdeca4eee2.jpeg#averageHue=%23ececec&clientId=u3a44d191-03af-4&from=ui&id=u053ea1c3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=322155&status=done&style=none&taskId=u96e29fdc-f781-4eb7-9b49-cb88cc871a9&title=)

`global 3`表示increase the number of bits to examine，现在查看前三个bits：

![69.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202213113-c419ae25-9e9c-4882-bc1c-6ae055fa3c85.jpeg#averageHue=%23ebebeb&clientId=u3a44d191-03af-4&from=ui&id=u0e59adba&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=321886&status=done&style=none&taskId=u39aa749d-b001-4d3c-99c9-24e66e69c33&title=)

![70.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202215524-78f60764-dacf-4c10-97a4-8245b7dcf533.jpeg#averageHue=%23ebeaea&clientId=u3a44d191-03af-4&from=ui&id=u508cc7e3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=356153&status=done&style=none&taskId=ua185fc1c-c364-402f-8e41-b8b8b032a22&title=)

![71.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202217586-585d3400-3552-4624-8682-6c4a521f8bf9.jpeg#averageHue=%23ebeaea&clientId=u3a44d191-03af-4&from=ui&id=u6295d532&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=364732&status=done&style=none&taskId=u686473d1-0303-465f-8d96-33e8d0ef274&title=)

![72.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202225915-9b6596db-dd53-4729-87f7-4001e180fa32.jpeg#averageHue=%23ebeaea&clientId=u3a44d191-03af-4&from=ui&id=u21c91758&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=386647&status=done&style=none&taskId=ubfac0162-2a8c-4aa0-bba8-f3128cecca5&title=)

## Linear Hashing

Instead of immediately splitting a bucket when it overflows, this scheme maintains a split pointer that keeps track of the next bucket to split. No matter whether this pointer is pointing to a bucket that overflowed, the DBMS always splits. The overflow criterion is left up to the implementation.

- When any bucket overflows, split the bucket at the pointer location by adding a new slot entry, and create a new hash function.
- If the hash function maps to slot that has previously been pointed to by pointer, apply the new hash function.
- When the pointer reaches last slot, delete original hash function and replace it with a new hash function.

上述Extendible Hashing是指数级的扩张hash表的，这未免也太快了。所以便有了LINEAR HASHING，线性扩张hash表，即每次只增加一个。其基本思想是维护一个指针split pointer，其指向下一个被拆分bucket，当任意bucket溢出时，将指针指向的当前bucket的拆分。

![73.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202225512-fb572d16-dd62-4041-a425-6416742465eb.jpeg#averageHue=%23ececec&clientId=u3a44d191-03af-4&from=ui&id=uc0390b2c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=314584&status=done&style=none&taskId=u1c880ccc-fd68-479c-9dc5-c0d3e491f1d&title=)

### 线性哈希的数学原理

假定key = 5 、 9 、13

```Plain
key % 4 = 1
```

现在我们对8求余

```Plain
5 % 8 = 5
9 % 8=1
13 % 8 = 5
```

由上面的规律可以得出

> (任意key) % n = M
> (任意key) %2n = M 或 (任意key) %2n = M + n

### 线性哈希的具体实现

我们假设初始化的哈希表如下:

|分裂点|桶编号|桶中已存储的Key|溢出key|
| ------| ------| -----------------| -------|
|*|0|4, 8, 12||
||1|5, 9||
||2|6||
||3|7, 11, 15, 19, 23||

为了方便叙述，我们作出以下假定:

1. 为了使哈希表能进行动态的分裂，我们从桶0开始设定一个分裂点。
2. 一个桶的容量为`listSize = 5`，当**桶的容量超出后就从分裂点开始进行分裂**。
3. hash函数为 $h0 = key \%4$,  $h1 = key \% 8$，h1会在分裂时使用。
4. 整个表初始化包含了4个桶，桶号为0-3，并已提前插入了部分的数据。

分裂过程如下，现在插入key = 27：

1. 进行哈希运算，$h0 = 27 \% 4 = 3$
2. 将key = 27插入桶3，但发现桶3已经达到了桶的容量，所以触发哈希分裂
3. 由于现在分裂点处于0桶，所以我们对0桶进行分割。这里需要注意虽然这里是**3桶满了，但我们并不会直接从3桶进行分割，而是从分割点进行分割**。这里为什么这么做，在下面会进一步介绍。
4. 对分割点所指向的桶(桶0)所包含的key采用新的hash函数($h1$)进行分割。

对所有key进行新哈希函数运算后，将产生如下的哈希表

|分裂点|桶编号|桶中已存储的Key|溢出key|
| ------| ------| -----------------| -------|
||0|8||
|*|1|5,9||
||2|6||
||3|7, 11, 15, 19, 23|27|
||4|4，12||

1. 虽然进行了分裂，**但桶3并不是分裂点，所以桶3会将多出的key，放于溢出页**,一直等到桶3进行分裂。
2. 进行分裂后，将分裂点向后移动一位。

一次完整的分裂结束。

**key的读取**：
采用$h0$对key进行计算。**如果算出的桶号小于了分裂点，表示桶已经进行的分裂**，我们采用$h1$进行hash运算，算出key所对应的真正的桶号。再从真正的桶里取出value，如果算出的桶号大于了分裂点，那么表示此桶还没进行分裂，直接从当前桶进行读取value。

**说明:**

1. 如果下一次key插入0、1、2、4桶，是不会触发分裂。（没有超出桶的容量）如果是插入桶3，用户在实现时可以自己设定，可以一旦插入就触发，也可以**等溢出页达到**​`**listSize**`​**再触发新的分裂**。
2. 现在0桶被分裂了，新数据的插入怎么才能保证没分裂的桶能正常工作，已经分裂的桶能将部分插入到新分裂的桶呢?

- 只要分裂点小于桶的总数，我们依然采用$h0$函数进行哈希计算。
- 如果哈希结果小于分裂号，那么表示这个key所插入的桶已经进行了分割，那么我就采用$h1$再次进行哈希，而$h1$的哈希结果就这个key所该插入的桶号。
- 如果哈希结果大于分裂号，那么表示这个key所插入的桶还没有进行分裂。直接插入。
- 这也是为什么虽然是桶3的容量不足，但分裂的桶是**分裂点所指向的桶**。如果直接在桶3进行分裂，那么当新的key插入的时候就不能正常的判断哪些桶已经进行了分裂。

1. 如果使用分割点，就具备了无限扩展的能力。**当分割点移动到最后一个桶(桶3)。再出现分裂。那么分割点就会回到桶0**，到这个时候，$h0$作废，$h1$替代$h0$，$h2(key \% 12)$替代$h1$。那么又可以开始动态分割。那个整个初始化状态就发生了变化。就好像没有发生过分裂。那么上面的规则就可以循环使用。

线性哈希的论文中是按上面的规则来进行分裂的。其实我们可以安装自己的实际情况来进行改动。
假如我们现在希望去掉分割点，一旦哪个桶满了，马上对这个桶进行分割。
可以考虑了以下方案：

1. 为所有桶增加一个标志位。初始化的时候对所有桶的标志位清空。
2. 一旦某个桶满了，直接对这个桶进行分割，然后将设置标志位。当新的数据插入的时候，经过哈希计算（$h0$）发现这个桶已经分裂了，那么就采用新的哈希函数($h1$)来计算分裂之后的桶号。在读取数据的时候处理类似。

![74.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202226162-6d77e9ce-5df8-43b4-a5c5-0d44a78a1da9.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=u33226d0d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=295865&status=done&style=none&taskId=u4a295a3e-7e92-4047-b922-5b79baed9c7&title=)

![75.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202228749-92755f36-0ffc-4300-89e7-6338c0883589.jpeg#averageHue=%23edecec&clientId=u3a44d191-03af-4&from=ui&id=u48709773&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=316214&status=done&style=none&taskId=ue91c9098-4fe6-4a97-a5d2-93c922cd9fc&title=)

![76.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202231755-d63e3033-811b-4b42-a5e6-96ee6a0ff5a0.jpeg#averageHue=%23ecebeb&clientId=u3a44d191-03af-4&from=ui&id=u30b986e6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=339344&status=done&style=none&taskId=uc62696cf-394f-4b93-b790-9cadf28de8e&title=)

![77.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202241210-81730154-1b9c-44b1-bfba-d461749fa270.jpeg#averageHue=%23ecebeb&clientId=u3a44d191-03af-4&from=ui&id=u4026e99c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=338770&status=done&style=none&taskId=u652cdcfb-f349-4143-852f-215309d15e5&title=)

![78.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202241217-99568a73-de28-4e3e-a690-dea2316169a9.jpeg#averageHue=%23eceaea&clientId=u3a44d191-03af-4&from=ui&id=uc4c912af&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=361996&status=done&style=none&taskId=u6efdd0f3-57f4-4d47-abfe-734061fba31&title=)

![79.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202241221-38bef652-a82f-4d64-b006-076a01c3c7a0.jpeg#averageHue=%23eae9e9&clientId=u3a44d191-03af-4&from=ui&id=u0c9db7d8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=375428&status=done&style=none&taskId=u3b09a42e-22a8-444e-8e72-b956ca57370&title=)

![80.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202244957-16fdef2f-18b2-45b7-bcfe-5b78fc33637f.jpeg#averageHue=%23eae9e8&clientId=u3a44d191-03af-4&from=ui&id=u6a185fc5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=402525&status=done&style=none&taskId=u516a748d-b72e-4e24-b771-7ad1bc70ded&title=)

![81.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202247589-a9c8eb1d-33fc-435f-b1ae-6db91356d870.jpeg#averageHue=%23eae9e9&clientId=u3a44d191-03af-4&from=ui&id=u766f92cc&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=430711&status=done&style=none&taskId=u697d2721-0514-4006-8654-61d3f9397e6&title=)

![82.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202257206-6accf439-5a20-45ae-90d0-cffd535b42ed.jpeg#averageHue=%23eae8e8&clientId=u3a44d191-03af-4&from=ui&id=ud7cd3bed&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=450721&status=done&style=none&taskId=u05baffe1-2fc8-4f75-845d-b09a9132333&title=)

![83.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202250106-4a148597-46a6-4dfa-b1ed-9a6ef8747b6b.jpeg#averageHue=%23eeeeee&clientId=u3a44d191-03af-4&from=ui&id=u6bc38062&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=250631&status=done&style=none&taskId=u5ffbe449-2aa6-4125-9cae-90bc64d0ec8&title=)

![84.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202254796-19cb8956-7cc1-4cc1-b9f1-69aef24d5c95.jpeg#averageHue=%23eae9e9&clientId=u3a44d191-03af-4&from=ui&id=u7da25a27&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=346812&status=done&style=none&taskId=uc1282b16-e5cd-4592-b1d2-1c9b795c391&title=)

![85.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202259188-6f048719-990a-4d0b-b13a-a7f6c4e84c27.jpeg#averageHue=%23eae8e8&clientId=u3a44d191-03af-4&from=ui&id=u74b4edd0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=376092&status=done&style=none&taskId=u681307f5-76ec-4f8c-9b63-5d02d0ff142&title=)

![86.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202262797-53c78e1b-5fc0-4988-b060-3c0153cfeda3.jpeg#averageHue=%23eae9e9&clientId=u3a44d191-03af-4&from=ui&id=u44e10504&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=398352&status=done&style=none&taskId=ubc0ce935-eb03-4f69-ab19-2e81ef42631&title=)

![87.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202265884-1d021010-9fa6-48b0-a80b-0263fb8e8593.jpeg#averageHue=%23ebe9e9&clientId=u3a44d191-03af-4&from=ui&id=ua6fe526d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=370589&status=done&style=none&taskId=uf46bc092-aaeb-4982-866b-ba7a1c8dfac&title=)

![88.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202274219-119f7441-790e-4e5f-9cc5-7bd4dca9dd28.jpeg#averageHue=%23eae9e9&clientId=u3a44d191-03af-4&from=ui&id=uef208da3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=364252&status=done&style=none&taskId=u9a554c5f-55be-4c07-b686-b063724559f&title=)

![89.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202275441-cde974cc-a33f-46a7-8e5d-ed265cac9480.jpeg#averageHue=%23ebeaea&clientId=u3a44d191-03af-4&from=ui&id=uf145785e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=339258&status=done&style=none&taskId=u430300e5-f32c-47d4-8e4b-58654e60a04&title=)

![90.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202280464-6912b4db-9c0e-410d-80aa-411039e5bc62.jpeg#averageHue=%23ebeaea&clientId=u3a44d191-03af-4&from=ui&id=u83e73967&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=373013&status=done&style=none&taskId=u80cbea00-02b2-4db9-8386-4039baae813&title=)

![91.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202280622-b407d134-cf38-40d7-9313-3144eabfceae.jpeg#averageHue=%23eeeeee&clientId=u3a44d191-03af-4&from=ui&id=u90a078ac&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=262554&status=done&style=none&taskId=u3e79d482-9a64-461f-912c-e55438237a0&title=)

![92.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678202279014-0a02804c-548a-497e-92d1-7a4b282dbc51.jpeg#averageHue=%23f0f0f0&clientId=u3a44d191-03af-4&from=ui&id=u6b03987c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=162036&status=done&style=none&taskId=u0d52c605-b308-4326-9790-148cd98852f&title=)
