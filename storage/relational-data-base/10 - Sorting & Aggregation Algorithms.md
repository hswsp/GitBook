# 10 - Sorting & Aggregation Algorithms

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645257207-c5babf81-bbd4-4ba2-a06d-f3cafa18e2b9.jpeg#averageHue=%231d2635&clientId=u53a3e605-e96c-4&from=ui&id=u43f43f61&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=602572&status=done&style=none&taskId=u555d3e82-bd88-4c2c-b913-e20cd635a50&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645256091-9953bd67-fc24-49ee-bfea-44e9c9680e60.jpeg#averageHue=%23ededed&clientId=u53a3e605-e96c-4&from=ui&id=u6b88d2a2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=309642&status=done&style=none&taskId=uf73a41a1-4250-4a3f-b3c5-a52a5f7cc06&title=)

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645256242-044987bd-3727-422f-b51f-c851910a83cb.jpeg#averageHue=%23dddddd&clientId=u53a3e605-e96c-4&from=ui&id=u59bf0ff7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=372786&status=done&style=none&taskId=u17c29198-83ab-4d2c-9150-e6f36afa90a&title=)

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645256139-81470248-5397-4219-9818-680da12344e2.jpeg#averageHue=%23e5e5e5&clientId=u53a3e605-e96c-4&from=ui&id=u111a0e25&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=352145&status=done&style=none&taskId=u8761c8bc-9437-4032-bdd7-c75e6747edb&title=)

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645256209-ba7d6081-61ab-41c6-989f-ed28da546b88.jpeg#averageHue=%23ebebeb&clientId=u53a3e605-e96c-4&from=ui&id=ue96d1b89&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=339018&status=done&style=none&taskId=udf566a9f-3c2f-4914-bd6d-3f71d469463&title=)

# Sorting

DBMSs need to sort data because tuples in a table have no specific order under the relation model. Sorting is (potentially) used in `ORDER BY`, `GROUP BY`, `JOIN`, and `DISTINCT` operators. If the data that that needs to be sorted fits in memory, then the DBMS can use a standard sorting algorithms (e.g., quicksort). If the data does not fit, then the DBMS needs to use external sorting that is able to spill to disk as needed and prefers sequential over random I/O.

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645259433-da0fa109-4e22-4bc3-ba81-9e74fdf8aa21.jpeg#averageHue=%23ebebeb&clientId=u53a3e605-e96c-4&from=ui&id=ua691d891&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=358635&status=done&style=none&taskId=uc3af2a54-9157-47a8-99b0-50751af5119&title=)

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645259015-cb608a4d-dff5-4867-adcc-554eb83d9dd8.jpeg#averageHue=%23ededed&clientId=u53a3e605-e96c-4&from=ui&id=u3be7c1b9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=275758&status=done&style=none&taskId=u74d8cbe7-26c5-475d-8ee8-3d537a63d9f&title=)

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645258869-d5ac1de1-a463-4642-b986-144c585dfd9f.jpeg#averageHue=%23f0f0f0&clientId=u53a3e605-e96c-4&from=ui&id=ueb3919b2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=174163&status=done&style=none&taskId=u3c1b9f71-b9e3-487b-9acf-db735795dfd&title=)

If a query contains an `ORDER BY` with a `LIMIT`, then the DBMS only needs to scan the data once to find the top-N elements. This is called the Top-N Heap Sort. The ideal scenario for heapsort is when the top-N elements fit in memory, so that the DBMS only has to maintain an in-memory sorted priority queue while scanning the data.

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645260258-baad6c4c-7ec9-4c17-ae70-a6450295e599.jpeg#averageHue=%23e7e6e6&clientId=u53a3e605-e96c-4&from=ui&id=u42b4d654&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=398021&status=done&style=none&taskId=u1dc35e49-750e-4802-ae1a-962a17d3bc4&title=)

The standard algorithm for sorting data which is too large to fit in memory is external merge sort. It is a divide-and-conquer sorting algorithm that splits the data set into separate *runs* and then sorts them individually. It can spill runs to disk as needed then read them back in one at a time. The algorithm is comprised of two phases:
**Phase #1 – Sorting**: First, the algorithm sorts small chunks of data that fit in main memory, and then writes the sorted pages back to disk.
**Phase #2# – Merge: **Then, the algorithm combines the sorted sub-files into a larger single file.

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645260803-22a7b9fe-892e-4662-816f-585c418dc3aa.jpeg#averageHue=%23ebebeb&clientId=u53a3e605-e96c-4&from=ui&id=u4594ddf9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=333670&status=done&style=none&taskId=u500eb637-9c36-46fa-9376-6bdb990246e&title=)

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645264443-a4021059-f628-4db2-99fd-5882478287a9.jpeg#averageHue=%23eaeaea&clientId=u53a3e605-e96c-4&from=ui&id=ufa5ef5a5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=332227&status=done&style=none&taskId=u1bf776e7-8b62-4d15-a7c9-6c50fff225b&title=)

## Two-way Merge Sort

The most basic version of the algorithm is the two-way merge sort. The algorithm reads each page during the sorting phase, sorts it, and writes the sorted version back to disk. Then, in the merge phase, it uses three buffer pages. It reads two sorted pages in from disk, and merges them together into a third buffer page. Whenever the third page fills up, it is written back to disk and replaced with an empty page. Each set of sorted pages is called a run. The algorithm then recursively merges the runs together.
If $N$ is the total number of data pages, the algorithm makes $1 + ⌈ log_2 N ⌉$ total passes through the data (1 for the first sorting step then $⌈ log_2 N ⌉$ for the recursive merging). The total I/O cost is $2N × (\# of \enspace passes)$ since each pass performs an I/O read and an I/O write for each page.

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645264969-42030626-0ad4-4488-992d-b63a7bb0db3d.jpeg#averageHue=%23ececec&clientId=u53a3e605-e96c-4&from=ui&id=u8f7f1ea3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=315152&status=done&style=none&taskId=u5fce3487-fcd4-4cf7-b26e-84dfef9e8cd&title=)

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645266489-466683cb-e601-4d26-95b3-80e866e1051a.jpeg#averageHue=%23e6e6e6&clientId=u53a3e605-e96c-4&from=ui&id=uf364e2df&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=402730&status=done&style=none&taskId=u6fcf8eea-ae0b-4272-9e35-dfe534ed296&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645267398-ea2f4f6c-ce86-4ed7-9b9b-5ba6288ffacb.jpeg#averageHue=%23eaeaea&clientId=u53a3e605-e96c-4&from=ui&id=ubaab669a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=388418&status=done&style=none&taskId=u0e66ecf2-8231-46a4-922e-79747fa2320&title=)

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645267192-1361d09f-0a0b-46f3-b4f1-e7ccd9a9824b.jpeg#averageHue=%23ececec&clientId=u53a3e605-e96c-4&from=ui&id=u4de05aea&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=309308&status=done&style=none&taskId=u00a96de2-d512-41ec-a892-aa29c528926&title=)

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645268480-6a364bfc-a294-4045-a8ab-25a8bc775656.jpeg#averageHue=%23eaeaea&clientId=u53a3e605-e96c-4&from=ui&id=ub6932236&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=326827&status=done&style=none&taskId=u185e535b-87de-4516-ab3c-28082f108da&title=)

## General (K-way) Merge Sort

The generalized version of the algorithm allows the DBMS to take advantage of using more than three buffer pages. Let $B$ be the total number of buffer pages available. Then, during the sort phase, the algorithm can read B pages at a time and write $\lceil \frac{N}{B} \rceil$ sorted runs back to disk. The merge phase can also combine up to $B − 1$ runs in each pass, again using one buffer page for the combined data and writing back to disk as needed.
In the generalized version, the algorithm performs $1 + log_{B−1}{ \lceil \frac{N}{B} \rceil }$passes (one for the sorting phase and $log_{B−1}{ \lceil \frac{N}{B} \rceil }$ for the merge phase. Then, the total I/O cost is $2N × (\# of \enspace passes)$ since it again has to make a read and write for each page in each pass.

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645268507-0057fa12-3bd6-4dcd-92b3-63ac9a3f1b06.jpeg#averageHue=%23eeeded&clientId=u53a3e605-e96c-4&from=ui&id=u4223f08d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=273982&status=done&style=none&taskId=uab0190f1-5558-4c62-a9e7-cf880738293&title=)

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645269720-40511d1c-96b0-4773-a4c8-1ff034e276f6.jpeg#averageHue=%23ececec&clientId=u53a3e605-e96c-4&from=ui&id=ud78f3e33&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=381117&status=done&style=none&taskId=u6ee599ea-f507-40d1-87e6-cfede2b16d0&title=)

## Double Buffering Optimization

One optimization for external merge sort is prefetching the next run in the background and storing it in a second buffer while the system is processing the current run. This reduces the wait time for I/O requests at each step by continuously utilizing the disk. This optimization requires the use of multiple threads, since the prefetching should occur while the computation for the current run is happening.

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645270786-cdf72b0a-a947-457c-8672-3827518627e7.jpeg#averageHue=%23ebebeb&clientId=u53a3e605-e96c-4&from=ui&id=u3a32dc85&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=352481&status=done&style=none&taskId=u8f821bd3-c67b-4b57-b032-4627ca64722&title=)

## Using B+Trees

It is sometimes advantageous for the DBMS to use an existing B+tree index to aid in sorting rather than using the external merge sort algorithm. In particular, if the index is a clustered index, the DBMS can just traverse the B+tree. Since the index is clustered, the data will be stored in the correct order, so the I/O access will be sequential. This means it is always better than external merge sort since no computation is required. On the other hand, if the index is unclustered, traversing the tree is almost always worse, since each record could be stored in any page, so nearly all record accesses will require a disk read.

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645271102-ce33a2f9-1163-4f54-b318-f50d095dabc3.jpeg#averageHue=%23ebebeb&clientId=u53a3e605-e96c-4&from=ui&id=ud2020653&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319792&status=done&style=none&taskId=u4d3700d5-cd81-4e15-a0ca-eaf2f503be3&title=)

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645272098-a6b745ac-5ca4-4502-8a60-786b9dc11b7e.jpeg#averageHue=%23e8e7e7&clientId=u53a3e605-e96c-4&from=ui&id=uff8d04a7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=359376&status=done&style=none&taskId=u40349d45-fdf7-4650-9184-8502302c97a&title=)

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645271934-67679865-6914-4d3f-bae7-403ab02235ae.jpeg#averageHue=%23e9e8e8&clientId=u53a3e605-e96c-4&from=ui&id=u61efbf15&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=316560&status=done&style=none&taskId=ucc8aae15-3b2c-4632-ac95-da45f68f28b&title=)

# Aggregations

An aggregation operator in a query plan collapses the values of one or more tuples into a single scalar value. There are two approaches for implementing an aggregation: (1) sorting and (2) hashing.

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645272710-69aa21d1-345c-484a-b3d1-db132046242b.jpeg#averageHue=%23ededed&clientId=u53a3e605-e96c-4&from=ui&id=u50a22bed&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=292158&status=done&style=none&taskId=ua7056104-050a-43a1-a49f-8446545197b&title=)

## Sorting

The DBMS first sorts the tuples on the `GROUP BY key(s)`. It can use either an in-memory sorting algorithm if everything fits in the buffer pool (e.g., quicksort) or the external merge sort algorithm if the size of the data exceeds memory. The DBMS then performs a sequential scan over the sorted data to compute the aggregation. The output of the operator will be sorted on the keys.
When performing sorting aggregations, it is important to order the query operations to maximize efficiency. For example, if the query requires a filter, it is better to perform the filter first and then sort the filtered data to reduce the amount of data that needs to be sorted.

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645273839-ac504ad2-e356-4f35-bfca-78fbb6fc93f5.jpeg#averageHue=%23e0dfdf&clientId=u53a3e605-e96c-4&from=ui&id=ua66ee454&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=431421&status=done&style=none&taskId=u5b0a3413-38a8-4d98-9a38-7ee2bf028f5&title=)

## Hashing

Hashing can be computationally cheaper than sorting for computing aggregations. The DBMS populates an ephemeral hash table as it scans the table. For each record, check whether there is already an entry in the hash table and perform the appropriate modification. If the size of the hash table is too large to fit in memory, then the DBMS has to spill it to disk. There are two phases to accomplishing this:
• **Phase #1 – Partition:**  Use a hash function $h_1$ to split tuples into partitions on disk based on target hash key. This will put all tuples that match into the same partition. The DBMS spills partitions to disk via output buffers.
• **Phase #2 – ReHash**: For each partition on disk, read its pages into memory and build an in-memory hash table based on a second hash function $h_2$ (where $h_1 \neq h_2$ ). Then go through each bucket of this hash table to bring together matching tuples to compute the aggregation. This assumes that each partition fits in memory.

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645274279-2145c944-9786-44a3-91b6-9265638afd2f.jpeg#averageHue=%23ececec&clientId=u53a3e605-e96c-4&from=ui&id=u8cf0d931&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=315359&status=done&style=none&taskId=u5cf1f4bc-bffa-43ac-8af9-d045646ed74&title=)

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645277352-5a68a0dc-0285-4810-aa14-f5056707f0ea.jpeg#averageHue=%23ebebeb&clientId=u53a3e605-e96c-4&from=ui&id=uc7feddb4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=351005&status=done&style=none&taskId=u2ee6b579-e971-4b2e-a735-49557cc4d94&title=)

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645275918-98d4ef45-8b0a-4e4a-ad38-a7ddc647b4db.jpeg#averageHue=%23ededed&clientId=u53a3e605-e96c-4&from=ui&id=udb2a78d5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=268902&status=done&style=none&taskId=u74aa587a-f95a-42bf-af6c-0fa10828e48&title=)

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645277907-a2e14445-ac1e-42fe-8688-97ec462b55a6.jpeg#averageHue=%23ececec&clientId=u53a3e605-e96c-4&from=ui&id=u075882ad&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=322798&status=done&style=none&taskId=uba416639-dbdb-4536-832a-62a21ea294b&title=)

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645283753-a9e56aba-6a33-4125-822f-57a1ee653d42.jpeg#averageHue=%23e2e1e1&clientId=u53a3e605-e96c-4&from=ui&id=udce41452&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=416761&status=done&style=none&taskId=u6e0bf42b-4a3b-4c99-a35c-f63ab13336f&title=)

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645281521-aef91c43-c5e0-40f9-b65b-7003c590e211.jpeg#averageHue=%23ededed&clientId=u53a3e605-e96c-4&from=ui&id=ud9404e07&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=282573&status=done&style=none&taskId=uec759245-0f2c-4f58-be3e-4c3ede1bbe4&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645287845-0fe118a6-9c09-4971-9fe6-43af0fa06e67.jpeg#averageHue=%23e4e3e3&clientId=u53a3e605-e96c-4&from=ui&id=u2cc168dc&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=431108&status=done&style=none&taskId=u92b26c80-5159-4042-b831-bf661db1542&title=)

During the ReHash phase, the DBMS can store pairs of the form (`GroupByKey`→`RunningValue`) to compute the aggregation. The contents of `RunningValue` depends on the aggregation function. To insert a new tuple into the hash table:
• If it finds a matching `GroupByKey`, then update the `RunningValue` appropriately.
• Else insert a new (`GroupByKey`→`RunningValue`) pair.

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645286708-da0f5ba9-602a-4d77-83ca-7610496848ba.jpeg#averageHue=%23ededed&clientId=u53a3e605-e96c-4&from=ui&id=u8f3479d0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=313910&status=done&style=none&taskId=ub6f7ff34-7637-44cc-b606-5fbfc27c9ea&title=)

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645289454-51cce759-be14-456c-95e0-b7b5a3ac6a4d.jpeg#averageHue=%23e2e1e1&clientId=u53a3e605-e96c-4&from=ui&id=u1faf881d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=447520&status=done&style=none&taskId=uaf0976ad-75c7-4644-bded-95aee5531ea&title=)

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645289451-50cb78d3-ea15-45c7-b2d0-5448e40155cf.jpeg#averageHue=%23ededed&clientId=u53a3e605-e96c-4&from=ui&id=u2fba22e1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=276585&status=done&style=none&taskId=ubffa28be-5c6d-451e-9fd8-c3993f279c1&title=)

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1679645289282-0bac8917-bf13-4215-b65f-74e0643fa4ea.jpeg#averageHue=%23f0f0f0&clientId=u53a3e605-e96c-4&from=ui&id=u169a14d4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=161279&status=done&style=none&taskId=u49cb6359-009b-4ab4-8453-3560dc97b06&title=)
