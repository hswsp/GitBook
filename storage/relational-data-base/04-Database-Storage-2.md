# 04 - Database Storage 2

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071120522-651a3878-583a-4c89-b2b5-7428251032a2.jpeg#averageHue=%231d2635&clientId=ub22c0555-07f7-4&from=ui&id=u5eccdd5e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=605612&status=done&style=none&taskId=u447a5e00-413c-4daa-a777-f0897316f35&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071119754-b38b1eb3-b433-476a-96b4-8f746fb33cd8.jpeg#averageHue=%23efefef&clientId=ub22c0555-07f7-4&from=ui&id=ubd1db22a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=233937&status=done&style=none&taskId=ub9d36786-8bc2-4494-891c-fde3518d291&title=)

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071120236-c9f1b2ef-9d6a-45ab-a0c7-bd3fbf0e149f.jpeg#averageHue=%23efebe7&clientId=ub22c0555-07f7-4&from=ui&id=u866e36c8&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=479906&status=done&style=none&taskId=ufe9c8766-3ed6-4c71-a5fd-3cb973cc3ee&title=)

# Log-Structured Storage

Some problems associated with the Slotted-Page Design are:

- Fragmentation: Deletion of tuples can leave gaps in the pages.
- Useless Disk I/O: Due to the block-oriented nature of non-volatile storage, the whole block needs to be read to fetch a tuple.
- Random Disk I/O: The disk reader could have to jump to 20 different places to update 20 different tuples, which can be very slow.

What if we were working on a system which only allows creation of new data and no overwrites? The log-structured storage model works with this assumption and addresses some of the problems listed above.

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071119914-eeb8ded9-f285-43d1-8753-2a2559ff24c4.jpeg#averageHue=%23ececec&clientId=ub22c0555-07f7-4&from=ui&id=uad622f24&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=277361&status=done&style=none&taskId=u52a2cf3f-dbc8-4b58-a484-04add9965ba&title=)

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071120197-ae13f57c-8a94-469a-85ba-5c4fd63b7074.jpeg#averageHue=%23eaeaea&clientId=ub22c0555-07f7-4&from=ui&id=uffc970df&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=375715&status=done&style=none&taskId=u99e682bc-896b-4a0c-8eb4-19cb799206f&title=)

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071122059-0cde7e0e-969e-42d9-9a6b-5373219808fa.jpeg#averageHue=%23e9e9e9&clientId=ub22c0555-07f7-4&from=ui&id=ufcd6ab96&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=348841&status=done&style=none&taskId=uc7c8f3f3-4dd8-4947-8ec6-2335f940ef0&title=)

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071121723-cffbae37-4281-4647-bb85-cb2bba05aa0e.jpeg#averageHue=%23f0f0f0&clientId=ub22c0555-07f7-4&from=ui&id=ua0771279&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=180835&status=done&style=none&taskId=uf8e28243-306f-41da-b3c6-8702d99e103&title=)

**Log-Structured Storage**: Instead of storing tuples, the DBMS only stores log records.

- Stores records to ﬁle of how the database was modiﬁed (put and delete). Each log record contains the tuple’s unique identiﬁer.
- To read a record, the DBMS scans the log ﬁle backwards from newest to oldest and “recreates” the tuple.
- Fast writes, potentially slow reads. Disk writes are sequential and existing pages are immutable which leads to reduced random disk I/O.
- Works well on append-only storage because the DBMS cannot go back and update the data.
- To avoid long reads, the DBMS can have **indexes** to allow it to jump to speciﬁc locations in the log. It can also periodically compact the log. (If it had a tuple and then made an update to it, it could compact it down to just inserting the updated tuple.)
- The database can compact the log into a table **sorted by the id **since the temporal information is not needed anymore. These are called Sorted String Tables (SSTables) and they can make the tuple search very fast.
- **The issue with compaction is that the DBMS ends up with write ampliﬁcation**. (It re-writes the same data over and over again.)

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071123056-3782e870-9675-40a7-aec6-0041840070a9.jpeg#averageHue=%23e8e8e8&clientId=ub22c0555-07f7-4&from=ui&id=u387f93e1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=479369&status=done&style=none&taskId=ufe4de63e-b8cd-4c0a-979c-8ebc9bdc230&title=)

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071122655-8a475ad1-1255-407a-9e26-c2b7c6509a6c.jpeg#averageHue=%23ebebeb&clientId=ub22c0555-07f7-4&from=ui&id=u9c95f315&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=334834&status=done&style=none&taskId=u077f526b-2af6-4a68-a310-e239fcfa807&title=)

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071123439-91c77b00-c3f9-4d20-ba35-60fbbe6ef0d7.jpeg#averageHue=%23e6e5e5&clientId=ub22c0555-07f7-4&from=ui&id=uc77c69e7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=549424&status=done&style=none&taskId=u9c8a1637-9e1b-4d07-8549-dad3c6b1025&title=)

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071124209-50087c67-eb9b-4107-8dcf-9ebdca8016b5.jpeg#averageHue=%23e6e5e5&clientId=ub22c0555-07f7-4&from=ui&id=ue099086e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=549424&status=done&style=none&taskId=u2f49a66f-0b08-4fa0-9a06-a16271aa57c&title=)

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071124592-a4f45da2-b6bd-4273-abcc-944a49c994ca.jpeg#averageHue=%23eaeaea&clientId=ub22c0555-07f7-4&from=ui&id=u9d231f1c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=449065&status=done&style=none&taskId=u9e82dc4a-0df0-4867-9e10-e647220e2a4&title=)

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071124992-a640aa3e-4c92-4559-af35-1cdafa82a92e.jpeg#averageHue=%23e9e8e8&clientId=ub22c0555-07f7-4&from=ui&id=u2b877a1a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=437762&status=done&style=none&taskId=ue2fa9d0e-3276-4d6b-bf36-3d1ceb345f5&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071125505-a341d10b-7d4c-44fe-9553-0debb9eb12b8.jpeg#averageHue=%23e7e7e7&clientId=ub22c0555-07f7-4&from=ui&id=uce548cbe&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=444487&status=done&style=none&taskId=u3782fbaf-8d44-4817-8bf6-06a3c63feab&title=)

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071125597-fc7f8d68-4f5e-4486-a63c-0cec821b7374.jpeg#averageHue=%23edecec&clientId=ub22c0555-07f7-4&from=ui&id=u6617bba3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=318985&status=done&style=none&taskId=u7d08151c-2591-42a7-8f32-02a51491c7f&title=)

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071126473-3d7efe02-d660-4450-b96f-f605c7cc2d18.jpeg#averageHue=%23ebebeb&clientId=ub22c0555-07f7-4&from=ui&id=ua89da875&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=398119&status=done&style=none&taskId=uf3e1b5b9-b709-4e63-a5df-1e6b96ad154&title=)

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071126724-bfe67ebb-d263-41d4-8b1c-1665e5674cad.jpeg#averageHue=%23ececeb&clientId=ub22c0555-07f7-4&from=ui&id=u4c3f6fc0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=343122&status=done&style=none&taskId=u08f3d219-8913-4176-82ef-356b6f15904&title=)

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071127290-f4b2da9c-7d40-4e7d-9bc6-66ac9249ba24.jpeg#averageHue=%23ededed&clientId=ub22c0555-07f7-4&from=ui&id=u2a2d03ea&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=293346&status=done&style=none&taskId=u8f28b1e6-f2b0-4642-9891-7bafe2ce676&title=)

# Data Representation

The data in a tuple is essentially just byte arrays. It is up to the DBMS to know how to interpret those bytes to derive the values for attributes. A ***data representation***** scheme** is how a DBMS stores the bytes for a value.
There are ﬁve high level datatypes that can be stored in tuples: integers, variable-precision numbers, ﬁxed-point precision numbers, variable length values, and dates/times.

## Integers

Most DBMSs store integers using their “native” C/C++ types as speciﬁed by the IEEE-754 standard. These values are ﬁxed length.
Examples: `INTEGER`, `BIGINT`, `SMALLINT`, `TINYINT`.

## Variable Precision Numbers

These are inexact, variable-precision numeric types that use the “native” C/C++ types speciﬁed by IEEE-754 standard. These values are also ﬁxed length.
Operations on variable-precision numbers are faster to compute than arbitrary precision numbers because the CPU can execute instructions on them directly. However, there may be rounding errors when performing computations due to the fact that some numbers cannot be represented precisely.
Examples: `FLOAT`, `REAL`.

## Fixed-Point Precision Numbers

These are numeric data types **with arbitrary precision and scale**. They are typically stored in exact, variable-length binary representation (almost like a string) with additional meta-data that will tell the system things like the length of the data and where the decimal should be.
These data types are used when rounding errors are unacceptable, but the DBMS pays a performance penalty to get this accuracy.
Examples: `NUMERIC`, `DECIMAL`.

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071127595-a842e0b6-dd7d-4306-b047-cdde2e79f1b4.jpeg#averageHue=%23edecec&clientId=ub22c0555-07f7-4&from=ui&id=u579bdd0b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=395509&status=done&style=none&taskId=u783fd7b0-4368-490b-8e5d-4c28cb5a12c&title=)

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071127727-69b44d0b-386a-47b1-a28a-447921f9220e.jpeg#averageHue=%23ececec&clientId=ub22c0555-07f7-4&from=ui&id=u9ddb7f70&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=303870&status=done&style=none&taskId=uca1c8345-a6cd-4652-afd6-37b4a910ec6&title=)

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071128955-d4abadd0-9819-4458-92c4-df4c55ff9529.jpeg#averageHue=%23ededed&clientId=ub22c0555-07f7-4&from=ui&id=u80cda99f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=305414&status=done&style=none&taskId=ue7d52a49-6b06-4a1b-80e0-29cc259f5ae&title=)

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071129270-326e3981-2e2d-4d66-a73f-806320613103.jpeg#averageHue=%23ececec&clientId=ub22c0555-07f7-4&from=ui&id=uee3b1fd6&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=408528&status=done&style=none&taskId=uf25c6eff-9d5b-436e-b1a1-d04b1a63bba&title=)

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071129602-b7fc9c6b-f9aa-4c25-9203-2ddfd95598d3.jpeg#averageHue=%23ebebeb&clientId=ub22c0555-07f7-4&from=ui&id=ufc01031e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=349992&status=done&style=none&taskId=udb458f8a-4fc3-431f-b2ad-921c339127b&title=)

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071130073-be249e44-f700-40c8-8b65-3369476ba857.jpeg#averageHue=%23efefef&clientId=ub22c0555-07f7-4&from=ui&id=u3c2ec5b5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=436073&status=done&style=none&taskId=u71d0cf0f-e2a4-4988-85a3-b53755e6e95&title=)

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071130082-bcf490b9-728e-4468-bd48-e789a9769d98.jpeg#averageHue=%23eeeded&clientId=ub22c0555-07f7-4&from=ui&id=ub7650230&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=349482&status=done&style=none&taskId=u891db899-e97d-4ca8-b9c6-3587c468c56&title=)

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071131418-9751d0e3-992b-4729-af05-ee2021f36034.jpeg#averageHue=%23e6e9e7&clientId=ub22c0555-07f7-4&from=ui&id=u3e16405c&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=368909&status=done&style=none&taskId=u0c6dea6f-ef91-41cd-ac94-12ad7156ee9&title=)

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071131408-17ea76fb-0aef-43dd-bccd-aadef3bc3ab2.jpeg#averageHue=%23eeeded&clientId=ub22c0555-07f7-4&from=ui&id=uc0eea83a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=336049&status=done&style=none&taskId=u4570d128-16d5-4e43-9fed-c388b3f265c&title=)

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071131680-5cae0d7b-476a-4813-9338-e081cb742347.jpeg#averageHue=%23f8f7f7&clientId=ub22c0555-07f7-4&from=ui&id=uc39ff20b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=368324&status=done&style=none&taskId=u696a7c32-db23-4e7c-87f4-2f74a9a8950&title=)

# Variable-Length Data

These represent data types of arbitrary length. They are typically stored with a **header** that keeps track of the length of the string to make it easy to jump to the next value. It may also contain a **checksum** for the data.
Most DBMSs do not allow a tuple to exceed the size of a single page. The ones that do store the data on a special “overﬂow” page and have the tuple contain a reference to that page. These *overﬂow pages can contain pointers* to additional overﬂow pages until all the data can be stored.
Some systems will let you store these large values in an **external ﬁle**, and then the tuple will contain a pointer to that ﬁle. For example, if the database is storing photo information, the DBMS can store the photos in the external ﬁles rather than having them take up large amounts of space in the DBMS. One downside of this is that the DBMS cannot manipulate the contents of this ﬁle. Thus, there are **no durability or transaction protections**.
Examples: `VARCHAR`, `VARBINARY`, `TEXT`, `BLOB`.

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071132197-c1033ae3-0367-4c20-9ab3-5f41eab9195c.jpeg#averageHue=%23ebebeb&clientId=ub22c0555-07f7-4&from=ui&id=u5b70bf21&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=375335&status=done&style=none&taskId=u4d43488e-ca6b-44c7-a630-8de8d3787b5&title=)

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071132418-4024ed78-cfea-4505-8fef-05e946b93a23.jpeg#averageHue=%23eaeaea&clientId=ub22c0555-07f7-4&from=ui&id=uabae3674&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=378764&status=done&style=none&taskId=u462b447c-6c7c-4e4e-aecc-83e7a09c19a&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071133971-07eae942-2e8c-4a1e-991f-e8df2decb1c2.jpeg#averageHue=%23eaeaea&clientId=ub22c0555-07f7-4&from=ui&id=uad3a6476&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=649464&status=done&style=none&taskId=u1f6d8651-3169-47cb-9c77-21ae842cf4d&title=)

# Dates and Times

Representations for date/time vary for different systems. Typically, these are represented as some unit time (micro/milli)seconds since the unix epoch.
Examples: `TIME`, `DATE`, `TIMESTAMP`.

# System Catalogs

In order for the DBMS to be able to decipher the contents of tuples, it maintains an internal **catalog to tell it meta-data about the databases.**  The meta-data will contain information about what tables and columns the databases have along with their types and the orderings of the values.
Most DBMSs store their catalog inside of themselves in the format that they use for their tables. They use special code to “bootstrap” these catalog tables.

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071133678-4c5ec88f-5aeb-45ec-8c25-15b5381a69dc.jpeg#averageHue=%23ececec&clientId=ub22c0555-07f7-4&from=ui&id=u703d5734&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=326815&status=done&style=none&taskId=u783adaf4-fae6-4563-8ecc-0aee035ba78&title=)

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071133934-7019962d-781d-437b-b071-b9207bfcaf46.jpeg#averageHue=%23ececec&clientId=ub22c0555-07f7-4&from=ui&id=u611e4f4d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=323254&status=done&style=none&taskId=uc88341a4-dc43-4ec7-bb45-3613cbcb7b2&title=)

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071134386-32a34f10-3ffc-47f1-a4a3-0b027ed5072f.jpeg#averageHue=%23edecec&clientId=ub22c0555-07f7-4&from=ui&id=u59c8856f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=312612&status=done&style=none&taskId=u9d3d6a1e-5d8b-48d5-98a9-d2b434e38d2&title=)

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071134679-dc0af686-5575-449b-b9e6-bfbdb1633b50.jpeg#averageHue=%23ececec&clientId=ub22c0555-07f7-4&from=ui&id=ua4509b4d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319845&status=done&style=none&taskId=uf0555bab-6cff-4dc9-8a24-c711faf4799&title=)

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1678071135508-17252bfc-f999-4d83-bc1e-eb0dee5bf927.jpeg#averageHue=%23eeeeee&clientId=ub22c0555-07f7-4&from=ui&id=u6fda954d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=256671&status=done&style=none&taskId=u9ca7027b-3169-4beb-90f4-d833d2762e0&title=)
