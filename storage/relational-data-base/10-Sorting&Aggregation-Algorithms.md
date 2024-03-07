# 10 - Sorting & Aggregation Algorithms

![1.jpg](1679645257207-c5babf81-bbd4-4ba2-a06d-f3cafa18e2b9.jpeg)

![2.jpg](1679645256091-9953bd67-fc24-49ee-bfea-44e9c9680e60.jpeg)

![3.jpg](assets/1679645256242-044987bd-3727-422f-b51f-c851910a83cb.jpeg)

![4.jpg](assets/1679645256139-81470248-5397-4219-9818-680da12344e2.jpeg)

![5.jpg](assets/1679645256209-ba7d6081-61ab-41c6-989f-ed28da546b88.jpeg)

# Sorting

DBMSs need to sort data because tuples in a table have no specific order under the relation model. Sorting is (potentially) used in `ORDER BY`, `GROUP BY`, `JOIN`, and `DISTINCT` operators. If the data that that needs to be sorted fits in memory, then the DBMS can use a standard sorting algorithms (e.g., quicksort). If the data does not fit, then the DBMS needs to use external sorting that is able to spill to disk as needed and prefers sequential over random I/O.

![6.jpg](assets/1679645259433-da0fa109-4e22-4bc3-ba81-9e74fdf8aa21.jpeg)

![7.jpg](assets/1679645259015-cb608a4d-dff5-4867-adcc-554eb83d9dd8.jpeg)

![8.jpg](assets/1679645258869-d5ac1de1-a463-4642-b986-144c585dfd9f.jpeg)

If a query contains an `ORDER BY` with a `LIMIT`, then the DBMS only needs to scan the data once to find the top-N elements. This is called the Top-N Heap Sort. The ideal scenario for heapsort is when the top-N elements fit in memory, so that the DBMS only has to maintain an in-memory sorted priority queue while scanning the data.

![9.jpg](assets/1679645260258-baad6c4c-7ec9-4c17-ae70-a6450295e599.jpeg)

The standard algorithm for sorting data which is too large to fit in memory is external merge sort. It is a divide-and-conquer sorting algorithm that splits the data set into separate *runs* and then sorts them individually. It can spill runs to disk as needed then read them back in one at a time. The algorithm is comprised of two phases:
**Phase #1 – Sorting**: First, the algorithm sorts small chunks of data that fit in main memory, and then writes the sorted pages back to disk.
**Phase #2# – Merge: **Then, the algorithm combines the sorted sub-files into a larger single file.

![10.jpg](assets/1679645260803-22a7b9fe-892e-4662-816f-585c418dc3aa.jpeg)

![11.jpg](assets/1679645264443-a4021059-f628-4db2-99fd-5882478287a9.jpeg)

## Two-way Merge Sort

The most basic version of the algorithm is the two-way merge sort. The algorithm reads each page during the sorting phase, sorts it, and writes the sorted version back to disk. Then, in the merge phase, it uses three buffer pages. It reads two sorted pages in from disk, and merges them together into a third buffer page. Whenever the third page fills up, it is written back to disk and replaced with an empty page. Each set of sorted pages is called a run. The algorithm then recursively merges the runs together.
If $N$ is the total number of data pages, the algorithm makes $1 + ⌈ log_2 N ⌉$ total passes through the data (1 for the first sorting step then $⌈ log_2 N ⌉$ for the recursive merging). The total I/O cost is $2N × (\# of \enspace passes)$ since each pass performs an I/O read and an I/O write for each page.

![12.jpg](assets/1679645264969-42030626-0ad4-4488-992d-b63a7bb0db3d.jpeg)

![13.jpg](assets/1679645266489-466683cb-e601-4d26-95b3-80e866e1051a.jpeg)

![14.jpg](assets/1679645267398-ea2f4f6c-ce86-4ed7-9b9b-5ba6288ffacb.jpeg)

![15.jpg](assets/1679645267192-1361d09f-0a0b-46f3-b4f1-e7ccd9a9824b.jpeg)

![16.jpg](assets/1679645268480-6a364bfc-a294-4045-a8ab-25a8bc775656.jpeg)

## General (K-way) Merge Sort

The generalized version of the algorithm allows the DBMS to take advantage of using more than three buffer pages. Let $B$ be the total number of buffer pages available. Then, during the sort phase, the algorithm can read B pages at a time and write $\lceil \frac{N}{B} \rceil$ sorted runs back to disk. The merge phase can also combine up to $B − 1$ runs in each pass, again using one buffer page for the combined data and writing back to disk as needed.
In the generalized version, the algorithm performs $1 + log_{B−1}{ \lceil \frac{N}{B} \rceil }$passes (one for the sorting phase and $log_{B−1}{ \lceil \frac{N}{B} \rceil }$ for the merge phase. Then, the total I/O cost is $2N × (\# of \enspace passes)$ since it again has to make a read and write for each page in each pass.

![17.jpg](assets/1679645268507-0057fa12-3bd6-4dcd-92b3-63ac9a3f1b06.jpeg)

![18.jpg](assets/1679645269720-40511d1c-96b0-4773-a4c8-1ff034e276f6.jpeg)

## Double Buffering Optimization

One optimization for external merge sort is prefetching the next run in the background and storing it in a second buffer while the system is processing the current run. This reduces the wait time for I/O requests at each step by continuously utilizing the disk. This optimization requires the use of multiple threads, since the prefetching should occur while the computation for the current run is happening.

![19.jpg](assets/1679645270786-cdf72b0a-a947-457c-8672-3827518627e7.jpeg)

## Using B+Trees

It is sometimes advantageous for the DBMS to use an existing B+tree index to aid in sorting rather than using the external merge sort algorithm. In particular, if the index is a clustered index, the DBMS can just traverse the B+tree. Since the index is clustered, the data will be stored in the correct order, so the I/O access will be sequential. This means it is always better than external merge sort since no computation is required. On the other hand, if the index is unclustered, traversing the tree is almost always worse, since each record could be stored in any page, so nearly all record accesses will require a disk read.

![20.jpg](assets/1679645271102-ce33a2f9-1163-4f54-b318-f50d095dabc3.jpeg)

![21.jpg](assets/1679645272098-a6b745ac-5ca4-4502-8a60-786b9dc11b7e.jpeg)

![22.jpg](assets/1679645271934-67679865-6914-4d3f-bae7-403ab02235ae.jpeg)

# Aggregations

An aggregation operator in a query plan collapses the values of one or more tuples into a single scalar value. There are two approaches for implementing an aggregation: (1) sorting and (2) hashing.

![23.jpg](assets/1679645272710-69aa21d1-345c-484a-b3d1-db132046242b.jpeg)

## Sorting

The DBMS first sorts the tuples on the `GROUP BY key(s)`. It can use either an in-memory sorting algorithm if everything fits in the buffer pool (e.g., quicksort) or the external merge sort algorithm if the size of the data exceeds memory. The DBMS then performs a sequential scan over the sorted data to compute the aggregation. The output of the operator will be sorted on the keys.
When performing sorting aggregations, it is important to order the query operations to maximize efficiency. For example, if the query requires a filter, it is better to perform the filter first and then sort the filtered data to reduce the amount of data that needs to be sorted.

![24.jpg](assets/1679645273839-ac504ad2-e356-4f35-bfca-78fbb6fc93f5.jpeg)

## Hashing

Hashing can be computationally cheaper than sorting for computing aggregations. The DBMS populates an ephemeral hash table as it scans the table. For each record, check whether there is already an entry in the hash table and perform the appropriate modification. If the size of the hash table is too large to fit in memory, then the DBMS has to spill it to disk. There are two phases to accomplishing this:
• **Phase #1 – Partition:**  Use a hash function $h_1$ to split tuples into partitions on disk based on target hash key. This will put all tuples that match into the same partition. The DBMS spills partitions to disk via output buffers.
• **Phase #2 – ReHash**: For each partition on disk, read its pages into memory and build an in-memory hash table based on a second hash function $h_2$ (where $h_1 \neq h_2$ ). Then go through each bucket of this hash table to bring together matching tuples to compute the aggregation. This assumes that each partition fits in memory.

![25.jpg](assets/1679645274279-2145c944-9786-44a3-91b6-9265638afd2f.jpeg)

![26.jpg](assets/1679645277352-5a68a0dc-0285-4810-aa14-f5056707f0ea.jpeg)

![27.jpg](assets/1679645275918-98d4ef45-8b0a-4e4a-ad38-a7ddc647b4db.jpeg)

![28.jpg](assets/1679645277907-a2e14445-ac1e-42fe-8688-97ec462b55a6.jpeg)

![29.jpg](assets/1679645283753-a9e56aba-6a33-4125-822f-57a1ee653d42.jpeg)

![30.jpg](assets/1679645281521-aef91c43-c5e0-40f9-b65b-7003c590e211.jpeg)

![31.jpg](assets/1679645287845-0fe118a6-9c09-4971-9fe6-43af0fa06e67.jpeg)

During the ReHash phase, the DBMS can store pairs of the form (`GroupByKey`→`RunningValue`) to compute the aggregation. The contents of `RunningValue` depends on the aggregation function. To insert a new tuple into the hash table:
• If it finds a matching `GroupByKey`, then update the `RunningValue` appropriately.
• Else insert a new (`GroupByKey`→`RunningValue`) pair.

![32.jpg](assets/1679645286708-da0f5ba9-602a-4d77-83ca-7610496848ba.jpeg)

![33.jpg](assets/1679645289454-51cce759-be14-456c-95e0-b7b5a3ac6a4d.jpeg)

![34.jpg](assets/1679645289451-50cb78d3-ea15-45c7-b2d0-5448e40155cf.jpeg)

![35.jpg](assets/1679645289282-0bac8917-bf13-4215-b65f-74e0643fa4ea.jpeg)
