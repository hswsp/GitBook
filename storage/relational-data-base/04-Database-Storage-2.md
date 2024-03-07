# 04 - Database Storage 2

![1.jpg](assets/1678071120522-651a3878-583a-4c89-b2b5-7428251032a2.jpeg)

![2.jpg](assets/1678071119754-b38b1eb3-b433-476a-96b4-8f746fb33cd8.jpeg)

![3.jpg](assets/1678071120236-c9f1b2ef-9d6a-45ab-a0c7-bd3fbf0e149f.jpeg)

# Log-Structured Storage

Some problems associated with the Slotted-Page Design are:

- Fragmentation: Deletion of tuples can leave gaps in the pages.
- Useless Disk I/O: Due to the block-oriented nature of non-volatile storage, the whole block needs to be read to fetch a tuple.
- Random Disk I/O: The disk reader could have to jump to 20 different places to update 20 different tuples, which can be very slow.

What if we were working on a system which only allows creation of new data and no overwrites? The log-structured storage model works with this assumption and addresses some of the problems listed above.

![4.jpg](assets/1678071119914-eeb8ded9-f285-43d1-8753-2a2559ff24c4.jpeg)

![5.jpg](assets/1678071120197-ae13f57c-8a94-469a-85ba-5c4fd63b7074.jpeg)

![6.jpg](assets/1678071122059-0cde7e0e-969e-42d9-9a6b-5373219808fa.jpeg)

![7.jpg](assets/1678071121723-cffbae37-4281-4647-bb85-cb2bba05aa0e.jpeg)

**Log-Structured Storage**: Instead of storing tuples, the DBMS only stores log records.

- Stores records to ﬁle of how the database was modiﬁed (put and delete). Each log record contains the tuple’s unique identiﬁer.
- To read a record, the DBMS scans the log ﬁle backwards from newest to oldest and “recreates” the tuple.
- Fast writes, potentially slow reads. Disk writes are sequential and existing pages are immutable which leads to reduced random disk I/O.
- Works well on append-only storage because the DBMS cannot go back and update the data.
- To avoid long reads, the DBMS can have **indexes** to allow it to jump to speciﬁc locations in the log. It can also periodically compact the log. (If it had a tuple and then made an update to it, it could compact it down to just inserting the updated tuple.)
- The database can compact the log into a table **sorted by the id **since the temporal information is not needed anymore. These are called Sorted String Tables (SSTables) and they can make the tuple search very fast.
- **The issue with compaction is that the DBMS ends up with write ampliﬁcation**. (It re-writes the same data over and over again.)

![8.jpg](assets/1678071123056-3782e870-9675-40a7-aec6-0041840070a9.jpeg)

![9.jpg](assets/1678071122655-8a475ad1-1255-407a-9e26-c2b7c6509a6c.jpeg)

![10.jpg](assets/1678071123439-91c77b00-c3f9-4d20-ba35-60fbbe6ef0d7.jpeg)

![11.jpg](assets/1678071124209-50087c67-eb9b-4107-8dcf-9ebdca8016b5.jpeg)

![12.jpg](assets/1678071124592-a4f45da2-b6bd-4273-abcc-944a49c994ca.jpeg)

![13.jpg](assets/1678071124992-a640aa3e-4c92-4559-af35-1cdafa82a92e.jpeg)

![14.jpg](assets/1678071125505-a341d10b-7d4c-44fe-9553-0debb9eb12b8.jpeg)

![15.jpg](assets/1678071125597-fc7f8d68-4f5e-4486-a63c-0cec821b7374.jpeg)

![16.jpg](assets/1678071126473-3d7efe02-d660-4450-b96f-f605c7cc2d18.jpeg)

![17.jpg](assets/1678071126724-bfe67ebb-d263-41d4-8b1c-1665e5674cad.jpeg)

![18.jpg](assets/1678071127290-f4b2da9c-7d40-4e7d-9bc6-66ac9249ba24.jpeg)

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

![19.jpg](assets/1678071127595-a842e0b6-dd7d-4306-b047-cdde2e79f1b4.jpeg)

![20.jpg](assets/1678071127727-69b44d0b-386a-47b1-a28a-447921f9220e.jpeg)

![21.jpg](assets/1678071128955-d4abadd0-9819-4458-92c4-df4c55ff9529.jpeg)

![22.jpg](assets/1678071129270-326e3981-2e2d-4d66-a73f-806320613103.jpeg)

![23.jpg](assets/1678071129602-b7fc9c6b-f9aa-4c25-9203-2ddfd95598d3.jpeg)

![24.jpg](assets/1678071130073-be249e44-f700-40c8-8b65-3369476ba857.jpeg)

![25.jpg](assets/1678071130082-bcf490b9-728e-4468-bd48-e789a9769d98.jpeg)

![26.jpg](assets/1678071131418-9751d0e3-992b-4729-af05-ee2021f36034.jpeg)

![27.jpg](assets/1678071131408-17ea76fb-0aef-43dd-bccd-aadef3bc3ab2.jpeg)

![28.jpg](assets/1678071131680-5cae0d7b-476a-4813-9338-e081cb742347.jpeg)

# Variable-Length Data

These represent data types of arbitrary length. They are typically stored with a **header** that keeps track of the length of the string to make it easy to jump to the next value. It may also contain a **checksum** for the data.
Most DBMSs do not allow a tuple to exceed the size of a single page. The ones that do store the data on a special “overﬂow” page and have the tuple contain a reference to that page. These *overﬂow pages can contain pointers* to additional overﬂow pages until all the data can be stored.
Some systems will let you store these large values in an **external ﬁle**, and then the tuple will contain a pointer to that ﬁle. For example, if the database is storing photo information, the DBMS can store the photos in the external ﬁles rather than having them take up large amounts of space in the DBMS. One downside of this is that the DBMS cannot manipulate the contents of this ﬁle. Thus, there are **no durability or transaction protections**.
Examples: `VARCHAR`, `VARBINARY`, `TEXT`, `BLOB`.

![29.jpg](assets/1678071132197-c1033ae3-0367-4c20-9ab3-5f41eab9195c.jpeg)

![30.jpg](assets/1678071132418-4024ed78-cfea-4505-8fef-05e946b93a23.jpeg)

![31.jpg](assets/1678071133971-07eae942-2e8c-4a1e-991f-e8df2decb1c2.jpeg)

# Dates and Times

Representations for date/time vary for different systems. Typically, these are represented as some unit time (micro/milli)seconds since the unix epoch.
Examples: `TIME`, `DATE`, `TIMESTAMP`.

# System Catalogs

In order for the DBMS to be able to decipher the contents of tuples, it maintains an internal **catalog to tell it meta-data about the databases.**  The meta-data will contain information about what tables and columns the databases have along with their types and the orderings of the values.
Most DBMSs store their catalog inside of themselves in the format that they use for their tables. They use special code to “bootstrap” these catalog tables.

![32.jpg](assets/1678071133678-4c5ec88f-5aeb-45ec-8c25-15b5381a69dc.jpeg)

![33.jpg](assets/1678071133934-7019962d-781d-437b-b071-b9207bfcaf46.jpeg)

![34.jpg](assets/1678071134386-32a34f10-3ffc-47f1-a4a3-0b027ed5072f.jpeg)

![35.jpg](assets/1678071134679-dc0af686-5575-449b-b9e6-bfbdb1633b50.jpeg)

![36.jpg](assets/1678071135508-17252bfc-f999-4d83-bc1e-eb0dee5bf927.jpeg)
