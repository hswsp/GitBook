# 05 - Columnar Databases & Compression

![1.jpg](assets/1678083489375-b66245c9-5f28-42e7-b663-967493714411.jpeg)

![2.jpg](assets/1678083488581-07e393f4-8939-4b2d-84cc-e775cd1a1e33.jpeg)

![3.jpg](assets/1678083488900-4300daae-c1cd-4919-ad64-723d07afcff0.jpeg)

## Database Workloads

### OLTP: Online Transaction Processing

An OLTP workload is characterized by fast, short running operations, simple queries that operate on single entity at a time, and repetitive operations. An OLTP workload will typically handle more writes than reads. An example of an OLTP workload is the Amazon storefront. Users can add things to their cart, they can make purchases, but the actions only affect their account.

### OLAP: Online Analytical Processing

An OLAP workload is characterized by long running, complex queries, reads on large portions of the database. In OLAP worklaods, the database system is analyzing and deriving new data from existing data collected on the OLTP side. An example of an OLAP workload would be Amazon computing the most bought item in Pittsburgh on a day when its raining.

### HTAP: Hybrid Transaction + Analytical Processing

A new type of workload which has become popular recently is HTAP, which is like a combination which tries to do OLTP and OLAP together on the same database.

![4.jpg](assets/1678083488851-2c6b7fe2-a391-4e75-b84d-c30c1551f413.jpeg)

![5.jpg](assets/1678083488969-f2ed8b47-31cc-4cbf-95fc-3e8b16030e37.jpeg)

![6.jpg](assets/1678083490369-ab99709b-9739-4caf-8f49-074deab5b147.jpeg)

![7.jpg](assets/1678083491065-b0ea3932-54d2-4642-8781-4a8fb0a7e759.jpeg)

![8.jpg](assets/1678083491676-6bb3f2a8-bc15-44bf-9c3a-fb4f73095ed8.jpeg)

## Storage Models

There are different ways to store tuples in pages. We have assumed the n-ary storage model so far.

![9.jpg](assets/1678083491527-38392b6c-27a2-4955-a94c-501789f25ec3.jpeg)

### N-Ary Storage Model (NSM)

In the n-ary storage model, the DBMS stores all of the attributes for a single tuple contiguously in a single page. This approach is ideal for **OLTP** workloads where requests are insert-heavy and transactions tend to operate only an individual entity. It is ideal because it takes only one fetch to be able to get all of the attributes for a single tuple.

#### Advantages:

* Fast inserts, updates, and deletes.
* Good for queries that need the entire tuple.

#### Disadvantages:

* Not good for scanning large portions of the table and/or a subset of the attributes.

![10.jpg](assets/1678083491526-6aba1203-1bab-4192-975a-444928cc0e43.jpeg)

![11.jpg](assets/1678083492471-fe757208-58b5-4d2f-9f1a-537f25129577.jpeg)

![12.jpg](assets/1678083493399-a395922c-a430-404f-bc26-d0e702185990.jpeg)

![13.jpg](assets/1678083493626-90387291-0eff-48b3-a0cc-4d6debbe7883.jpeg)

![14.jpg](assets/1678083493997-41cd9805-edaf-4621-a583-3deb29838ecc.jpeg)

![15.jpg](assets/1678083493760-abe71649-c904-470f-808d-3d07c886c619.jpeg)

### Decomposition Storage Model (DSM)

In the decomposition storage model, the DBMS stores a single attribute (column) for all tuples contiguously in a block of data. Thus, it is also known as a “column store.” This model is ideal for **OLAP** workloads with many read-only queries that perform large scans over a subset of the table’s attributes.

#### Advantages:

* Reduces the amount of I/O wasted because the DBMS only reads the data that it needs for that query.
* Better query processing and data compression

#### Disadvantages:

* Slow for point queries, inserts, updates, and deletes because of tuple splitting/stitching.

![16.jpg](assets/1678083494651-ceb635f4-129a-4fe4-854d-94031035593d.jpeg)

![17.jpg](assets/1678083495932-5ce57f14-9f40-4ace-b043-7581540b18ba.jpeg)

![18.jpg](assets/1678083496142-1713697f-6d41-44a0-939a-3f8238e869a0.jpeg)

![19.jpg](assets/1678083496473-2a05bde4-dc90-473a-9abe-bd33e7a2a619.jpeg)

To put the tuples back together when using a column store, there are two common approaches: The most commonly used approach is fixed-length offsets. Here, you know that the value in a given column will match to another value in another column at the same offset, they will correspond to the same tuple. Therefore, every single value within the column will have to be the same length. A less common approach is to use embedded tuple ids. Here, for every attribute in the columns, the DBMS stores a tuple id (ex: a primary key) with it. The system then would also store a mapping to tell it how to jump to every attribute that has that id. Note that this method has a large storage overhead because it needs to store a tuple id for every attribute entry.

![20.jpg](assets/1678083496558-ca77f716-e75d-42e8-98af-959ed3ad8515.jpeg)

![21.jpg](assets/1678083496706-a1f594e3-38a5-4a31-a12f-8b981a56050e.jpeg)

![22.jpg](assets/1678083498487-fe0e408b-84ea-4da9-97f1-84804d08ae32.jpeg)

## Database Compression

Compression is widely used in disk-based DBMSs. Because disk I/O is (almost) always the main bottleneck. Thus, compression in these systems improve performance, especially in read-only analytical workloads. The DBMS can fetch more useful tuples if they have been compressed beforehand at the cost of greater computational overhead for compression and decompression. In-memory DBMSs more complicated since they do not have to fetch data from disk to execute a query. Memory is much faster than disks, but compressing the database reduces DRAM requirements and processing. They have to strike a balance between **speed** vs. **compression** **ratio**. **Compressing the database reduces DRAM requirements.** It may decrease CPU costs during query execution.

![23.jpg](assets/1678083498433-db56bdee-f225-4907-9e36-a9d21bd8aea8.jpeg)

If data sets are completely random bits, there would be no ways to perform compression. However, there are key properties of real-world data sets that are amenable to compression:

* Data sets tend to have highly _skewed_ distributions for attribute values (e.g., Zipfian distribution of the Brown Corpus).
* Data sets tend to have high _correlation_ between attributes of the same tuple (e.g., Zip Code to City, Order Date to Ship Date).

![24.jpg](assets/1678083498698-4edf48d4-0b7d-4a51-b499-1da353583a1b.jpeg)

Given this, we want a database compression scheme to have the following properties:

* Must **produce fixed-length values**. The only exception is var-length data stored in separate pools. This because the DBMS should follow word-alignment and be able to access data using offsets.
* Allow the DBMS to **postpone decompression** as long as possible during query execution (late materialization).
* Must be a **lossless scheme** because people do not like losing data. Any kind of lossy compression has to be performed at the application level.

![25.jpg](assets/1678083498842-a50d9821-14c2-44d9-b741-3d16c35aafc4.jpeg)

![26.jpg](assets/1678083499963-a5d8eeba-1531-4769-9647-4453151a8a26.jpeg)

### Compression Granularity

Before adding compression to the DBMS, we need to decide what kind of data we want to compress. This decision determines compression schemes are available. There are four levels of compression granularity:

* **Block Level:** Compress a block of tuples for the same table.
* **Tuple Level:** Compress the contents of the entire tuple (NSM only).
* **Attribute Level:** Compress a single attribute value within one tuple. Can target multiple attributes for the same tuple.
* **Columnar Level:** Compress multiple values for one or more attributes stored for multiple tuples (DSM only). This allows for more complicated compression schemes.

![27.jpg](assets/1678083500847-ccf8fa97-28aa-42bf-a0cd-1a603fced7d1.jpeg)

## Naive Compression

The DBMS compresses data using a general purpose algorithm (e.g., gzip, LZO, LZ4, Snappy, Brotli, Oracle OZIP, Zstd). Although there are several compression algorithms that the DBMS could use, engineers often choose ones that often provides lower compression ratio in exchange for faster compress/decompress.

![28.jpg](assets/1678083500970-fc3a7774-25ea-4d36-a607-f17428d4a0c9.jpeg)

An example of using naive compression is in **MySQL InnoDB**. The DBMS compresses disk pages, pad them to a power of two KBs and stored them into the buffer pool. However, every time the DBMS tries to read data, the compressed data in the buffer pool has to be decompressed.

![29.jpg](assets/1678083501423-94560fbe-5f88-4c52-b77b-2ff2512cb37e.jpeg)

Since accessing data requires decompression of compressed data, this limits the scope of the compression scheme. If the goal is to compress the entire table into one giant block, using naive compression schemes would be impossible since the whole table needs to be compressed/decompressed for every access. Therefore, for MySQL, it breaks the table into smaller chunks since the compression scope is limited. Another problem is that these naive schemes also do not consider the high-level meaning or semantics of the data. The algorithm is oblivious to neither the structure of the data, nor how the query is planning to access the data. Therefore, this gives away the opportunity to utilize late materialization, since the DBMS will not be able to tell when it will be able to delay the decompression of data.

![30.jpg](assets/1678083501356-1a757b27-023b-48b9-ac44-2f7f60789508.jpeg)

![31.jpg](assets/1678083502045-9b46abfc-b759-4976-8d0f-3aaf9d7fb6ed.jpeg)

![32.jpg](assets/1678083503125-8987eae2-a6a5-4d2d-9909-9c16a910ca1f.jpeg)

## Columnar Compression

![33.jpg](assets/1678083502832-efcc67b2-f9de-4945-ab53-84e5f13b2662.jpeg)

### Run-Length Encoding (RLE)

RLE compresses runs of the same value in a single column into triplets:

* The value of the attribute
* The start position in the column segment
* The number of elements in the run

The DBMS should sort the columns intelligently beforehand to maximize compression opportunities. This clusters duplicate attributes and thereby increasing compression ratio.

![34.jpg](assets/1678083503471-63dde062-2afa-4360-8614-4ed8823ccce5.jpeg)

![35.jpg](assets/1678083503534-f4f34020-bd2a-422f-8989-b6ec7d3d567b.jpeg)

![36.jpg](assets/1678083503935-5118d505-34a3-4f0f-9713-a53092994f1b.jpeg)

![37.jpg](assets/1678083504717-e63c639d-1652-4c68-b315-c941d1f439b8.jpeg)

![38.jpg](assets/1678083504860-bc31de7c-2062-494f-83de-56daadae051a.jpeg)

### Bit-Packing Encoding

When values for an attribute are always less than the value’s declared largest size, store them as smaller data type.

![39.jpg](assets/1678083505173-a96cf5d4-a4ce-4e4b-87a9-a825f0325e7c.jpeg)

![40.jpg](assets/1678083505657-bb753980-a0a1-4cf0-8f25-a9a62bb16e13.jpeg)

### Mostly Encoding

Bit-packing variant that uses a special marker to indicate when a value exceeds largest size and then maintain a look-up table to store them.

![41.jpg](assets/1678083505784-fbedee71-d78c-416f-a46b-aea5f949b687.jpeg)

### Bitmap Encoding

The DBMS stores a separate bitmap for each unique value for a particular attribute where an offset in the vector corresponds to a tuple. The $i^{th}$ position in the bitmap corresponds to the $i^{th}$tuple in the table to indicate whether that value is present or not. The bitmap is **typically segmented into chunks** to avoid allocating large blocks of contiguous memory. **This approach is only practical if the value cardinality is low**, since the size of the bitmap is linear to the cardinality of the attribute value. If the cardinalty of the value is high, then the bitmaps can become larger than the original data set.

![42.jpg](assets/1678083507123-5dad5f67-357a-44fd-bfcd-7834d9f0f298.jpeg)

![43.jpg](assets/1678083507159-b0ffcb74-154c-4f28-88da-7b6f373f494a.jpeg)

![44.jpg](assets/1678083507281-5aad1006-a168-4773-b831-aa4b27b367c7.jpeg)

![45.jpg](assets/1678083508063-dd9af574-3d1b-408f-ba6f-a1784b26a805.jpeg)

### Delta Encoding

Instead of storing exact values, record the difference between values that follow each other in the same column. **The base value can be stored in-line or in a separate look-up table**. We can also use RLE on the stored deltas to get even better compression ratios.

![46.jpg](assets/1678083508330-9982038a-bf04-4c4e-a515-9b483282149a.jpeg)

![47.jpg](assets/1678083509559-4426defe-7bc4-41af-aeac-97768219ec2d.jpeg)

### Incremental Encoding

This is a type of delta encoding whereby common prefixes or suffixes and their lengths are recorded so that they need not be duplicated. This works best with sorted data.

![48.jpg](assets/1678083509638-e3521d08-84be-4f97-98fa-535b0d9ad703.jpeg)

### Dictionary Compression

The **most common** database compression scheme is dictionary encoding. The DBMS replaces frequent patterns in values with smaller codes. It then stores only these codes and a data structure (i.e., dictionary) that maps these codes to their original value. **A dictionary compression scheme needs to support fast encoding/decoding, as well as range queries.**

![49.jpg](assets/1678083509653-f181f482-9543-47bc-b504-33716937ad58.jpeg)

![50.jpg](assets/1678083510166-1d1592f4-96ed-4f53-b1cd-8a7babe7faaa.jpeg)

\*\*Encoding and Decoding: \*\*Finally, the dictionary needs to decide how to **encodes** (convert uncompressed value into its compressed form)/**decodes** (convert compressed value back into its original form) data. It is not possible to use hash functions.

![51.jpg](assets/1678083510644-06bd4002-c240-4203-9ec5-745a64dd8afd.jpeg)

The encoded values also need to **support sorting** in the same order as original values. This ensures that results returned for compressed queries run on compressed data are consistent with uncompressed queries run on original data. This **order-preserving property** allows operations to be performed directly on the codes.

![52.jpg](assets/1678083512304-62d2bb04-e442-4ac3-a3f3-a304392f50b1.jpeg)

![53.jpg](assets/1678083512286-3b7e36cc-7722-4b4e-be0c-0d28d3cbd136.jpeg)

![54.jpg](assets/1678083512172-dc393d21-4062-43c6-ab3e-03431ad0f417.jpeg)

![55.jpg](assets/1678083512253-dbfb9bd5-6817-4937-a7b8-722f7d518342.jpeg)
