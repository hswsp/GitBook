# 05 - Columnar Databases & Compression

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083489375-b66245c9-5f28-42e7-b663-967493714411.jpeg#averageHue=%231e2836&clientId=u9c241039-3215-4&from=ui&id=u8394d0a4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=614842&status=done&style=none&taskId=u25442d94-f008-471d-a7f0-f229bb28cdf&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083488581-07e393f4-8939-4b2d-84cc-e775cd1a1e33.jpeg#averageHue=%23f0efef&clientId=u9c241039-3215-4&from=ui&id=u72ccb27b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=200737&status=done&style=none&taskId=u1a8acb5e-5194-492c-9dfd-3ae98742d4f&title=)

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083488900-4300daae-c1cd-4919-ad64-723d07afcff0.jpeg#averageHue=%23eaeaea&clientId=u9c241039-3215-4&from=ui&id=u301e5f09&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=344835&status=done&style=none&taskId=u4a3d88f1-12c2-4699-8621-d2144e711f4&title=)

# Database Workloads

## OLTP: Online Transaction Processing

An OLTP workload is characterized by fast, short running operations, simple queries that operate on single entity at a time, and repetitive operations. An OLTP workload will typically handle more writes than reads.
An example of an OLTP workload is the Amazon storefront. Users can add things to their cart, they can make purchases, but the actions only affect their account.

## OLAP: Online Analytical Processing

An OLAP workload is characterized by long running, complex queries, reads on large portions of the database. In OLAP worklaods, the database system is analyzing and deriving new data from existing data collected on the OLTP side.
An example of an OLAP workload would be Amazon computing the most bought item in Pittsburgh on a day when its raining.

## HTAP: Hybrid Transaction + Analytical Processing

A new type of workload which has become popular recently is HTAP, which is like a combination which tries to do OLTP and OLAP together on the same database.

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083488851-2c6b7fe2-a391-4e75-b84d-c30c1551f413.jpeg#averageHue=%23e4dad9&clientId=u9c241039-3215-4&from=ui&id=u41309af4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=327573&status=done&style=none&taskId=uff21a7bd-3c03-471d-a593-5b889269863&title=)

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083488969-f2ed8b47-31cc-4cbf-95fc-3e8b16030e37.jpeg#averageHue=%23dcdcdc&clientId=u9c241039-3215-4&from=ui&id=ub322137a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=403556&status=done&style=none&taskId=uc322c79a-245a-4f1e-b286-60ceafddd5d&title=)

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083490369-ab99709b-9739-4caf-8f49-074deab5b147.jpeg#averageHue=%23eeeeee&clientId=u9c241039-3215-4&from=ui&id=u243ccfd0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=252814&status=done&style=none&taskId=u27bd336d-7998-4dba-8247-341a37215f6&title=)

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083491065-b0ea3932-54d2-4642-8781-4a8fb0a7e759.jpeg#averageHue=%23e4e4e4&clientId=u9c241039-3215-4&from=ui&id=u47e90cad&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=387195&status=done&style=none&taskId=uabd3a990-2d6c-453d-a9d8-037f369248c&title=)

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083491676-6bb3f2a8-bc15-44bf-9c3a-fb4f73095ed8.jpeg#averageHue=%23e8e8e8&clientId=u9c241039-3215-4&from=ui&id=ua4509cc3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=345876&status=done&style=none&taskId=ua3021437-9f09-4795-9811-dd455954588&title=)

# Storage Models

There are different ways to store tuples in pages. We have assumed the n-ary storage model so far.

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083491527-38392b6c-27a2-4955-a94c-501789f25ec3.jpeg#averageHue=%23ededed&clientId=u9c241039-3215-4&from=ui&id=u5a227acf&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=272945&status=done&style=none&taskId=u2f4e57a5-78b6-446e-8ad6-d908902bdba&title=)

## N-Ary Storage Model (NSM)

In the n-ary storage model, the DBMS stores all of the attributes for a single tuple contiguously in a single page. This approach is ideal for **OLTP** workloads where requests are insert-heavy and transactions tend to operate only an individual entity. It is ideal because it takes only one fetch to be able to get all of the attributes for a single tuple.

### Advantages:

- Fast inserts, updates, and deletes.
- Good for queries that need the entire tuple.

### Disadvantages:

- Not good for scanning large portions of the table and/or a subset of the attributes.

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083491526-6aba1203-1bab-4192-975a-444928cc0e43.jpeg#averageHue=%23ededed&clientId=u9c241039-3215-4&from=ui&id=u3198756b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=273263&status=done&style=none&taskId=u4e3a847a-3339-4da7-81ba-b2c9b5c66dc&title=)

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083492471-fe757208-58b5-4d2f-9f1a-537f25129577.jpeg#averageHue=%23eae9e9&clientId=u9c241039-3215-4&from=ui&id=ufed22d42&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=353732&status=done&style=none&taskId=uac4cf12d-09b5-43b7-b8e2-18867b6c421&title=)

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083493399-a395922c-a430-404f-bc26-d0e702185990.jpeg#averageHue=%23e5e5e5&clientId=u9c241039-3215-4&from=ui&id=ueb940db4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=368355&status=done&style=none&taskId=u36bf7e8d-7141-4fd4-8e8d-9141ba41c2d&title=)

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083493626-90387291-0eff-48b3-a0cc-4d6debbe7883.jpeg#averageHue=%23e3e2e2&clientId=u9c241039-3215-4&from=ui&id=u417ee2f0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=413974&status=done&style=none&taskId=u8d6d9535-1a8c-45fa-9543-c3b7ab17406&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083493997-41cd9805-edaf-4621-a583-3deb29838ecc.jpeg#averageHue=%23e1e0e0&clientId=u9c241039-3215-4&from=ui&id=u26de31da&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=450620&status=done&style=none&taskId=u92108cf7-29b2-4389-abd1-ecec358f3d8&title=)

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083493760-abe71649-c904-470f-808d-3d07c886c619.jpeg#averageHue=%23eeeeee&clientId=u9c241039-3215-4&from=ui&id=u4419ba87&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=250708&status=done&style=none&taskId=u980f077c-79a3-4e16-bcbb-afe211fb875&title=)

## Decomposition Storage Model (DSM)

In the decomposition storage model, the DBMS stores a single attribute (column) for all tuples contiguously in a block of data. Thus, it is also known as a “column store.” This model is ideal for **OLAP** workloads with many read-only queries that perform large scans over a subset of the table’s attributes.

### Advantages:

- Reduces the amount of I/O wasted because the DBMS only reads the data that it needs for that query.
- Better query processing and data compression

### Disadvantages:

- Slow for point queries, inserts, updates, and deletes because of tuple splitting/stitching.

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083494651-ceb635f4-129a-4fe4-854d-94031035593d.jpeg#averageHue=%23ebebeb&clientId=u9c241039-3215-4&from=ui&id=u582564f6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=307670&status=done&style=none&taskId=u908d6fec-9838-4404-8e03-7ad7ef96279&title=)

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083495932-5ce57f14-9f40-4ace-b043-7581540b18ba.jpeg#averageHue=%23e9e8e8&clientId=u9c241039-3215-4&from=ui&id=u71481df3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=345246&status=done&style=none&taskId=u28017640-613a-4773-9ba5-069313b38ad&title=)

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083496142-1713697f-6d41-44a0-939a-3f8238e869a0.jpeg#averageHue=%23e7e7e7&clientId=u9c241039-3215-4&from=ui&id=u1cec5b47&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=411505&status=done&style=none&taskId=u2f54601e-1916-42ef-8586-3e7f288b769&title=)

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083496473-2a05bde4-dc90-473a-9abe-bd33e7a2a619.jpeg#averageHue=%23e1e1e1&clientId=u9c241039-3215-4&from=ui&id=u7c45508b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=436433&status=done&style=none&taskId=u77b70671-c7b5-4449-b03f-e7673a86ff4&title=)

To put the tuples back together when using a column store, there are two common approaches: The most commonly used approach is fixed-length offsets. Here, you know that the value in a given column will match to another value in another column at the same offset, they will correspond to the same tuple. Therefore, every single value within the column will have to be the same length.
A less common approach is to use embedded tuple ids. Here, for every attribute in the columns, the DBMS stores a tuple id (ex: a primary key) with it. The system then would also store a mapping to tell it how to jump to every attribute that has that id. Note that this method has a large storage overhead because it needs to store a tuple id for every attribute entry.

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083496558-ca77f716-e75d-42e8-98af-959ed3ad8515.jpeg#averageHue=%23eae9e9&clientId=u9c241039-3215-4&from=ui&id=u3bab2c1a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=349225&status=done&style=none&taskId=u21dcb73f-6521-4548-85fd-0d6fd5bb215&title=)

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083496706-a1f594e3-38a5-4a31-a12f-8b981a56050e.jpeg#averageHue=%23ebebeb&clientId=u9c241039-3215-4&from=ui&id=u4798d849&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=314112&status=done&style=none&taskId=u804c7afc-ca4d-4e17-a41d-d2df5d202ae&title=)

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083498487-fe0e408b-84ea-4da9-97f1-84804d08ae32.jpeg#averageHue=%23eae9e8&clientId=u9c241039-3215-4&from=ui&id=uae82aee1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=531805&status=done&style=none&taskId=u730455d0-ba14-44d3-9858-82c2ec2bd30&title=)

# Database Compression

Compression is widely used in disk-based DBMSs. Because disk I/O is (almost) always the main bottleneck. Thus, compression in these systems improve performance, especially in read-only analytical workloads. The DBMS can fetch more useful tuples if they have been compressed beforehand at the cost of greater computational overhead for compression and decompression.
In-memory DBMSs more complicated since they do not have to fetch data from disk to execute a query. Memory is much faster than disks, but compressing the database reduces DRAM requirements and processing. They have to strike a balance between **speed** vs. **compression** **ratio**. **Compressing the database reduces DRAM requirements.**  It may decrease CPU costs during query execution.

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083498433-db56bdee-f225-4907-9e36-a9d21bd8aea8.jpeg#averageHue=%23ebebeb&clientId=u9c241039-3215-4&from=ui&id=u2bd206af&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=346742&status=done&style=none&taskId=ua6172600-3e58-41bb-ba34-eb69297c7a4&title=)

If data sets are completely random bits, there would be no ways to perform compression. However, there are key properties of real-world data sets that are amenable to compression:

- Data sets tend to have highly *skewed* distributions for attribute values (e.g., Zipfian distribution of the Brown Corpus).
- Data sets tend to have high *correlation* between attributes of the same tuple (e.g., Zip Code to City, Order Date to Ship Date).

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083498698-4edf48d4-0b7d-4a51-b499-1da353583a1b.jpeg#averageHue=%23ebebeb&clientId=u9c241039-3215-4&from=ui&id=u861e84c6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=317738&status=done&style=none&taskId=u9c47db23-726f-4ce0-9c00-25f662d1955&title=)

Given this, we want a database compression scheme to have the following properties:

- Must **produce fixed-length values**. The only exception is var-length data stored in separate pools. This because the DBMS should follow word-alignment and be able to access data using offsets.
- Allow the DBMS to **postpone decompression** as long as possible during query execution (late materialization).
- Must be a **lossless scheme** because people do not like losing data. Any kind of lossy compression has to be performed at the application level.

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083498842-a50d9821-14c2-44d9-b741-3d16c35aafc4.jpeg#averageHue=%23ececec&clientId=u9c241039-3215-4&from=ui&id=ucbe9269e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=312097&status=done&style=none&taskId=uc51cc9c5-d51e-41c8-a520-48d437846f5&title=)

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083499963-a5d8eeba-1531-4769-9647-4453151a8a26.jpeg#averageHue=%23ececec&clientId=u9c241039-3215-4&from=ui&id=ua790a2c8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=270061&status=done&style=none&taskId=u66e917ee-1b81-424a-9d71-fadeec7a4b2&title=)

## Compression Granularity

Before adding compression to the DBMS, we need to decide what kind of data we want to compress. This decision determines compression schemes are available. There are four levels of compression granularity:

- **Block Level:**  Compress a block of tuples for the same table.
- **Tuple Level:**  Compress the contents of the entire tuple (NSM only).
- **Attribute Level:**  Compress a single attribute value within one tuple. Can target multiple attributes for the same tuple.
- **Columnar Level:**  Compress multiple values for one or more attributes stored for multiple tuples (DSM only). This allows for more complicated compression schemes.

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083500847-ccf8fa97-28aa-42bf-a0cd-1a603fced7d1.jpeg#averageHue=%23e9e9e9&clientId=u9c241039-3215-4&from=ui&id=u4142468e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=417419&status=done&style=none&taskId=ua8b48d49-4791-4e20-96e8-b870fab1126&title=)

# Naive Compression

The DBMS compresses data using a general purpose algorithm (e.g., gzip, LZO, LZ4, Snappy, Brotli, Oracle OZIP, Zstd). Although there are several compression algorithms that the DBMS could use, engineers often choose ones that often provides lower compression ratio in exchange for faster compress/decompress.

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083500970-fc3a7774-25ea-4d36-a607-f17428d4a0c9.jpeg#averageHue=%23ededed&clientId=u9c241039-3215-4&from=ui&id=ucb9d96fe&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=316265&status=done&style=none&taskId=ufa8097f6-48c0-4bf3-9765-558986f681a&title=)

An example of using naive compression is in **MySQL InnoDB**. The DBMS compresses disk pages, pad them to a power of two KBs and stored them into the buffer pool. However, every time the DBMS tries to read data, the compressed data in the buffer pool has to be decompressed.

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083501423-94560fbe-5f88-4c52-b77b-2ff2512cb37e.jpeg#averageHue=%23e4e3e3&clientId=u9c241039-3215-4&from=ui&id=u139bb82c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=424226&status=done&style=none&taskId=udf94a330-c76c-4736-bddd-897828b27e4&title=)

Since accessing data requires decompression of compressed data, this limits the scope of the compression scheme. If the goal is to compress the entire table into one giant block, using naive compression schemes would be impossible since the whole table needs to be compressed/decompressed for every access. Therefore, for MySQL, it breaks the table into smaller chunks since the compression scope is limited.
Another problem is that these naive schemes also do not consider the high-level meaning or semantics of the data. The algorithm is oblivious to neither the structure of the data, nor how the query is planning to access the data. Therefore, this gives away the opportunity to utilize late materialization, since the DBMS will not be able to tell when it will be able to delay the decompression of data.

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083501356-1a757b27-023b-48b9-ac44-2f7f60789508.jpeg#averageHue=%23ededed&clientId=u9c241039-3215-4&from=ui&id=u52582f30&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=283518&status=done&style=none&taskId=u37c98b3d-c72a-4f47-8d9a-89ebdc5c5e2&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083502045-9b46abfc-b759-4976-8d0f-3aaf9d7fb6ed.jpeg#averageHue=%23e6e6e6&clientId=u9c241039-3215-4&from=ui&id=udc6e66c4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=332908&status=done&style=none&taskId=ufd0093c0-0cf6-4cfe-b7d8-c32246ec79b&title=)

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083503125-8987eae2-a6a5-4d2d-9909-9c16a910ca1f.jpeg#averageHue=%23e9e9e9&clientId=u9c241039-3215-4&from=ui&id=uf81ae69e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=418949&status=done&style=none&taskId=ua90c5b4a-2ac1-4e7a-ab39-744d22a9ce3&title=)

# Columnar Compression

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083502832-efcc67b2-f9de-4945-ab53-84e5f13b2662.jpeg#averageHue=%23eeeeee&clientId=u9c241039-3215-4&from=ui&id=ucb08f9d9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=234189&status=done&style=none&taskId=ued5a1f7e-d29e-45d8-9f3e-d80763a85ed&title=)

## Run-Length Encoding (RLE)

RLE compresses runs of the same value in a single column into triplets:

- The value of the attribute
- The start position in the column segment
- The number of elements in the run

The DBMS should sort the columns intelligently beforehand to maximize compression opportunities. This clusters duplicate attributes and thereby increasing compression ratio.

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083503471-63dde062-2afa-4360-8614-4ed8823ccce5.jpeg#averageHue=%23ececec&clientId=u9c241039-3215-4&from=ui&id=u53215390&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=296174&status=done&style=none&taskId=u68ea8577-3496-467e-9320-1784aa73f0d&title=)

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083503534-f4f34020-bd2a-422f-8989-b6ec7d3d567b.jpeg#averageHue=%23e8e7e7&clientId=u9c241039-3215-4&from=ui&id=u4156c0bd&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=291522&status=done&style=none&taskId=u3e13af12-0f2c-4475-967f-2f6e631f6fc&title=)

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083503935-5118d505-34a3-4f0f-9713-a53092994f1b.jpeg#averageHue=%23eaeae9&clientId=u9c241039-3215-4&from=ui&id=ub23291e3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=261063&status=done&style=none&taskId=u9e09d672-9241-4f25-803e-ac4b6d91017&title=)

![37.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083504717-e63c639d-1652-4c68-b315-c941d1f439b8.jpeg#averageHue=%23e8e7e7&clientId=u9c241039-3215-4&from=ui&id=u316661e6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=291343&status=done&style=none&taskId=u3c53095a-bf53-4526-aa18-915c82889ee&title=)

![38.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083504860-bc31de7c-2062-494f-83de-56daadae051a.jpeg#averageHue=%23eaeaea&clientId=u9c241039-3215-4&from=ui&id=u96d7613a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=245289&status=done&style=none&taskId=u21287e0d-2855-4186-9163-9ede26a1373&title=)

## Bit-Packing Encoding

When values for an attribute are always less than the value’s declared largest size, store them as smaller data type.

![39.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083505173-a96cf5d4-a4ce-4e4b-87a9-a825f0325e7c.jpeg#averageHue=%23ededed&clientId=u9c241039-3215-4&from=ui&id=u22a6e367&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=243531&status=done&style=none&taskId=u46b2f968-69d0-4343-94c5-67fee7166bd&title=)

![40.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083505657-bb753980-a0a1-4cf0-8f25-a9a62bb16e13.jpeg#averageHue=%23e9e9e9&clientId=u9c241039-3215-4&from=ui&id=u9e545e38&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=400897&status=done&style=none&taskId=u87a7365a-6abc-4f57-b98f-5e3a22bcad3&title=)

## Mostly Encoding

Bit-packing variant that uses a special marker to indicate when a value exceeds largest size and then maintain a look-up table to store them.

![41.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083505784-fbedee71-d78c-416f-a46b-aea5f949b687.jpeg#averageHue=%23eae9e9&clientId=u9c241039-3215-4&from=ui&id=u03cbfd9a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=368004&status=done&style=none&taskId=u5386d83c-8b24-413c-907e-fb585dc5b31&title=)

## Bitmap Encoding

The DBMS stores a separate bitmap for each unique value for a particular attribute where an offset in the vector corresponds to a tuple. The $i^{th}$ position in the bitmap corresponds to the $i^{th}$tuple in the table to indicate whether that value is present or not. The bitmap is **typically segmented into chunks** to avoid allocating large blocks of contiguous memory.
**This approach is only practical if the value cardinality is low**, since the size of the bitmap is linear to the cardinality of the attribute value. If the cardinalty of the value is high, then the bitmaps can become larger than the original data set.

![42.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083507123-5dad5f67-357a-44fd-bfcd-7834d9f0f298.jpeg#averageHue=%23ebebeb&clientId=u9c241039-3215-4&from=ui&id=ud010527b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=364195&status=done&style=none&taskId=u4bfabdd3-2a69-4b10-9df9-1a7bbfe35de&title=)

![43.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083507159-b0ffcb74-154c-4f28-88da-7b6f373f494a.jpeg#averageHue=%23e8e8e8&clientId=u9c241039-3215-4&from=ui&id=u73dae31c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=270588&status=done&style=none&taskId=u55e706e1-2a8c-439c-ba22-7b8113ffd22&title=)

![44.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083507281-5aad1006-a168-4773-b831-aa4b27b367c7.jpeg#averageHue=%23e8e7e7&clientId=u9c241039-3215-4&from=ui&id=u9334c820&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=315557&status=done&style=none&taskId=uaf41eea2-aa95-4470-abe8-4d390628738&title=)

![45.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083508063-dd9af574-3d1b-408f-ba6f-a1784b26a805.jpeg#averageHue=%23e4e4e4&clientId=u9c241039-3215-4&from=ui&id=uf35781ee&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=397967&status=done&style=none&taskId=ua0d03397-2fec-484d-b8b3-134f52d8066&title=)

## Delta Encoding

Instead of storing exact values, record the difference between values that follow each other in the same column. **The base value can be stored in-line or in a separate look-up table**. We can also use RLE on the stored deltas to get even better compression ratios.

![46.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083508330-9982038a-bf04-4c4e-a515-9b483282149a.jpeg#averageHue=%23e8e8e8&clientId=u9c241039-3215-4&from=ui&id=u6728113f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=393785&status=done&style=none&taskId=u580d6d3d-bf02-4240-a547-da96d00157d&title=)

![47.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083509559-4426defe-7bc4-41af-aeac-97768219ec2d.jpeg#averageHue=%23e8e7e7&clientId=u9c241039-3215-4&from=ui&id=ue5466784&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=440340&status=done&style=none&taskId=uacf63b93-457e-40ff-b39f-b501ecea448&title=)

## Incremental Encoding

This is a type of delta encoding whereby common prefixes or suffixes and their lengths are recorded so that they need not be duplicated. This works best with sorted data.

![48.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083509638-e3521d08-84be-4f97-98fa-535b0d9ad703.jpeg#averageHue=%23eae9e9&clientId=u9c241039-3215-4&from=ui&id=u67f6fcc8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=428895&status=done&style=none&taskId=u44d3e2c8-ef61-4992-ba8c-b72cfbbd602&title=)

## Dictionary Compression

The **most common** database compression scheme is dictionary encoding. The DBMS replaces frequent patterns in values with smaller codes. It then stores only these codes and a data structure (i.e., dictionary) that maps these codes to their original value. **A dictionary compression scheme needs to support fast encoding/decoding, as well as range queries.**

![49.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083509653-f181f482-9543-47bc-b504-33716937ad58.jpeg#averageHue=%23ebebeb&clientId=u9c241039-3215-4&from=ui&id=u19c7d69d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=345400&status=done&style=none&taskId=u8d8ac20f-a56c-4e3e-a69f-bfc2391b90c&title=)

![50.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083510166-1d1592f4-96ed-4f53-b1cd-8a7babe7faaa.jpeg#averageHue=%23e6e6e6&clientId=u9c241039-3215-4&from=ui&id=u3a438223&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=350492&status=done&style=none&taskId=ua46b7315-479b-4a17-93de-52a19d98a8f&title=)

**Encoding and Decoding: **Finally, the dictionary needs to decide how to **encodes** (convert uncompressed value into its compressed form)/**decodes** (convert compressed value back into its original form) data. It is not possible to use hash functions.

![51.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083510644-06bd4002-c240-4203-9ec5-745a64dd8afd.jpeg#averageHue=%23ececec&clientId=u9c241039-3215-4&from=ui&id=udf502616&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=294613&status=done&style=none&taskId=ub0f4ff26-73bc-462f-95a6-681a2432894&title=)

The encoded values also need to **support sorting** in the same order as original values. This ensures that results returned for compressed queries run on compressed data are consistent with uncompressed queries run on original data. This **order-preserving property** allows operations to be performed directly on the codes.

![52.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083512304-62d2bb04-e442-4ac3-a3f3-a304392f50b1.jpeg#averageHue=%23e4e4e3&clientId=u9c241039-3215-4&from=ui&id=u850ece02&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=421403&status=done&style=none&taskId=u06795290-592f-4270-9599-da00ecdd918&title=)

![53.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083512286-3b7e36cc-7722-4b4e-be0c-0d28d3cbd136.jpeg#averageHue=%23e4e3e3&clientId=u9c241039-3215-4&from=ui&id=ub25f0edd&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=424834&status=done&style=none&taskId=ub6f0c520-5d79-4b98-9b1e-15143e565c4&title=)

![54.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083512172-dc393d21-4062-43c6-ab3e-03431ad0f417.jpeg#averageHue=%23ececec&clientId=u9c241039-3215-4&from=ui&id=u40fbfe01&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319385&status=done&style=none&taskId=ua16acda3-2cb6-4215-9188-339bfd82f2b&title=)

![55.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678083512253-dbfb9bd5-6817-4937-a7b8-722f7d518342.jpeg#averageHue=%23ededed&clientId=u9c241039-3215-4&from=ui&id=u099be565&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=282974&status=done&style=none&taskId=u1b803520-973e-4bbc-9f9b-0c3d7086138&title=)
