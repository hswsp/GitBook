# 03 - Database Storage 1

![1.jpg](assets/1677726612688-2fb0193f-a5dd-492f-b922-a589daf5e425.jpeg)

![2.jpg](assets/1677726610528-17b02ef3-3ddb-4b9a-9016-cd9b2c8be601.jpeg)

![3.jpg](assets/1677726609607-8ccca72f-eca2-4d42-8b97-3c3d1d0f2e6e.jpeg)

# Storage

We will focus on a “disk-oriented” DBMS architecture that assumes that the primary storage location of the database is on non-volatile disk(s).

![4.jpg](assets/1677726610275-6cdb577c-10e4-4722-ab76-d85cdfff3535.jpeg)

![5.jpg](assets/1677726609622-6576a122-b1ef-473d-af5e-6a6cdbb89385.jpeg)

At the top of the storage hierarchy, you have the devices that are closest to the CPU. This is the fastest storage, but it is also the smallest and most expensive. The further you get away from the CPU, the larger but slower the storage devices get. These devices also get cheaper per GB.

![6.jpg](assets/1677726614303-93d131ae-1874-4e0d-a491-054e95f22f4c.jpeg)

**Volatile Devices:**

- Volatile means that if you pull the power from the machine, then the data is lost.
- **Volatile storage supports fast random access with byte-addressable locations.**  This means that the program can jump to any byte address and get the data that is there.
- For our purposes, we will always refer to this storage class as “memory.”

**Non-Volatile Devices:**

- Non-volatile means that the storage device does not require continuous power in order for the device to retain the bits that it is storing.
- It is also **block/page addressable.**  This means that in order to read a value at a particular offset, the program ﬁrst has to load the 4 KB page into memory that holds the value the program wants to read.
- Non-volatile storage is traditionally better at sequential access (reading multiple contiguous chunks of data at the same time).
- We will refer to this as “disk.” We will not make a (major) distinction between solid-state storage (SSD) and spinning hard drives (HDD).

![7.jpg](assets/1677726614683-98ed09cc-7d8a-44e6-96a5-2d7f194b35a2.jpeg)

There is also a relatively new class of storage devices that are becoming more popular called *persistent memory*. These devices are designed to be the best of both worlds: **almost as fast as DRAM with the persistence of disk**. We will not cover these devices in this course, and they are currently not in widespread production use. Probably the most famous example is Optane; unfortunately Intel is winding down its production as of summer 2022. Note that you may see older references to persistent memory as “non-volatile memory”.

![8.jpg](assets/1677726617088-c68d476a-b749-4c1f-9941-03adac740b6d.jpeg)

You may see references to NVMe SSDs, where NVMe stands for non-volatile memory express. These NVMe SSDs are not the same hardware as persistent memory modules. Rather, they are typical NAND ﬂash drives that connect over an improved hardware interface. This improved hardware interface allows for much faster transfers, which leverages improvements in NAND ﬂash perfomance.

![9.jpg](assets/1677726615015-ed8837ac-b6c5-49a6-b4ba-81f880efe732.jpeg)

Since our DBMS architecture assumes that the database is stored on disk, the components of the DBMS are responsible for ﬁguring out how to move data between non-volatile disk and volatile memory since the system cannot operate on the data directly on disk.
We will focus on hiding the latency of the disk rather than optimizations with registers and caches since getting data from disk is so slow. If reading data from the L1 cache reference took one second, reading from an SSD would take 4.4 hours, and reading from an HDD would take 3.3 weeks.

# Disk-Oriented DBMS Overview

The database is all on disk, and the data in database ﬁles is organized into pages, with the ﬁrst page being the **directory page**. To operate on the data, the DBMS needs to bring the data into memory. It does this by having a **buffer pool** that manages the data movement back and forth between disk and memory. The DBMS also has an **execution engine** that will execute queries. The execution engine will ask the buffer pool for a speciﬁc page, and the buffer pool will take care of bringing that page into memory and giving the execution engine a pointer to that page in memory. The buffer pool manager will ensure that the page is there while the execution engine operates on that part of memory.

![10.jpg](assets/1677726617332-9448b9a4-7d14-4176-84d2-c05de1ea4d86.jpeg)

![11.jpg](assets/1677726619249-0181a922-342b-4e07-a512-d98d34066c08.jpeg)

![12.jpg](assets/1677726619982-e04573da-553a-4946-ae47-c6d2f5dd5fd1.jpeg)

# DBMS vs. OS

A high-level design goal of the DBMS is to support databases that exceed the amount of memory available. Since reading/writing to disk is expensive, disk use must be carefully managed. We do not want large stalls from fetching something from disk to slow down everything else. We want the DBMS to be able to process other queries while it is waiting to get the data from disk.
This high-level design goal is like virtual memory, where there is a large address space and a place for the OS to bring in pages from disk.
One way to achieve this virtual memory is by using `mmap` to map the contents of a ﬁle in a process’ address space, which makes the OS responsible for moving pages back and forth between disk and memory. **Unfortunately, this means that if **​ **`**mmap**`**​ ** hits a page fault, the process will be blocked.**

- You never want to use `mmap` in your DBMS if you need to write.
- The DBMS (almost) always wants to control things itself and can do a better job at it since it knows more about the data being accessed and the queries being processed.
- The operating system is not your friend.

![13.jpg](assets/1677726620316-bbb97fec-a1cb-459a-b6b3-e7f21bde75cd.jpeg)

![14.jpg](assets/1677726622459-342c9439-0560-4d2e-b3d7-fa85365ea5f9.jpeg)

![15.jpg](assets/1677726622936-cf723fec-ee78-46a1-b579-2241d10ea9e1.jpeg)

![16.jpg](assets/1677726622911-5ccd102c-c851-4ddb-be5e-9081802a6336.jpeg)

![17.jpg](assets/1677726624885-184ab3bf-99ec-412a-b8b0-71876d9d4f1f.jpeg)

TLB ( Translation Lookaside Buffer。MMU 为了加速查找页表，使用的 cache。用来加速 MMU 的转化速度)是从虚拟内存地址到物理内存地址转换的高速缓存。当处理器更改地址的虚拟到物理映射时，它需要告诉其他处理器在其缓存中使该映射无效。这个过程被称为"[TLB shootdowns](https://juejin.cn/post/6844904084957315086)"。
It is possible to use the OS by using:

- `madvise`: Tells the OS know when you are planning on reading certain pages.
- `mlock`: Tells the OS to not swap memory ranges out to disk.
- `msync`: Tells the OS to ﬂush memory ranges out to disk.

![18.jpg](assets/1677726625569-e08aff22-d5d3-41ba-9d3e-582730ba316f.jpeg)

![19.jpg](assets/1677726626605-a4a85d7d-f028-4611-a876-6563a55b7cef.jpeg)

We do not advise using `mmap` in a DBMS for correctness and performance reasons.
Even though the system will have functionalities that seem like something the OS can provide, having the DBMS implement these procedures itself gives it better control and performance.

![20.jpg](assets/1677726629966-08d46fb2-18eb-4734-8ae7-142918b691ec.jpeg)

![21.jpg](assets/1677726627087-93c5a955-9be5-4fdd-a07c-6158b17db56a.jpeg)

![22.jpg](assets/1677726627651-28e94baf-2ca6-495a-82c8-709d4d98cbaa.jpeg)

# File Storage

In its most basic form, a DBMS stores a database as ﬁles on disk. Some may use a ﬁle hierarchy, others may use a single ﬁle (e.g., SQLite).
The OS does not know anything about the contents of these ﬁles. Only the DBMS knows how to decipher their contents, since it is encoded in a way speciﬁc to the DBMS.

![23.jpg](assets/1677726629502-b80231c6-d446-4d9b-8a31-6984b454df05.jpeg)

The DBMS’s *storage manager* is responsible for managing a database’s ﬁles. It represents the ﬁles **as a collection of pages**. It also keeps track of what data has been read and written to pages as well how much free space there is in these pages.

![24.jpg](assets/1677726630551-ac052419-c475-4ec5-ac97-adf3cdadf473.jpeg)

# Database Pages

The DBMS organizes the database across one or more ﬁles in ﬁxed-size blocks of data called pages. Pages can contain different kinds of data (tuples, indexes, etc). Most systems will not mix these types within pages. Some systems will require that pages are **self-contained**, meaning that all the information needed to read each page is on the page itself.
Each page is given a unique identiﬁer. If the database is a single ﬁle, then the page id can just be the ﬁle offset. Most DBMSs have an indirection layer that **maps a page id to a ﬁle path and offset**. The upper levels of the system will ask for a speciﬁc page number. Then, the storage manager will have to turn that page number into a ﬁle and an offset to ﬁnd the page.

![25.jpg](assets/1677726631159-2f0533ed-941e-4ecf-9a7c-cdcb179a3af3.jpeg)

Most DBMSs uses ﬁxed-size pages to avoid the engineering overhead needed to support variable-sized pages. For example, with variable-size pages, deleting a page could create a hole in ﬁles that the DBMS cannot easily ﬁll with new pages.
There are three concepts of pages in DBMS:

1. Hardware page (usually 4 KB).
2. OS page (4 KB).
3. Database page (1-16 KB).

**The storage device guarantees an atomic write of the size of the hardware page.**  If the hardware page is 4 KB and the system tries to write 4 KB to the disk, either all 4 KB will be written, or none of it will. This means that if our database page is larger than our hardware page, the DBMS will have to take extra measures to ensure that the data gets written out safely since the program can get partway through writing a database page to disk when the system crashes.

![26.jpg](assets/1677726632283-37ebae70-9b82-4768-bdab-929798a49595.jpeg)

# Database Heap

There are a couple of ways to ﬁnd the location of the page a DBMS wants on the disk, and **heap ﬁle** organization is one of those ways. A heap ﬁle is an unordered collection of pages where tuples are stored in random order.

![27.jpg](assets/1677726634356-1ad79dc0-f024-4318-b645-05523ed977a5.jpeg)

The DBMS can locate a page on disk given a page id by using a **linked list** of pages or a page directory.

1. ** Linked List**: Header page holds pointers to a list of free pages and a list of data pages. However, if the DBMS is looking for a speciﬁc page, it has to do a sequential scan on the data page list until it ﬁnds the page it is looking for.
2. **Page Directory**: DBMS maintains special pages that track locations of data pages along with the amount of free space on each page.

![28.jpg](assets/1677726635474-43c3f100-78c9-4b27-bd9e-07c6bdd77f9a.jpeg)

![29.jpg](assets/1677726635965-5f429e6c-19fd-4b1b-b6c2-de8acdb1f08a.jpeg)

![30.jpg](assets/1677726635960-6c76bf0e-5fff-45e9-a7fd-007ea1e4ca6f.jpeg)

![31.jpg](assets/1677726637542-8f0a72cd-c407-4466-b224-f2c34e17d94a.jpeg)

![32.jpg](assets/1677726637055-8de92311-17e3-480f-830c-48605a90f9d6.jpeg)

# Page Layout

Every page includes a header that records meta-data about the page’s contents:

- Page size.
- Checksum.
- DBMS version.
- Transaction visibility.
- Self-containment. (Some systems like Oracle require this.)

![33.jpg](assets/1677726639646-2f882de5-7c22-457c-920c-f333efaec637.jpeg)

A **strawman approach** to laying out data is to keep track of how many tuples the DBMS has stored in a page and then append to the end every time a new tuple is added. However, ***problems arise when tuples are deleted or when tuples have variable-length attributes***.
There are two **main approaches** to laying out data in pages: (1) **slotted-pages** and (2) **log-structured**.
**Slotted Pages:**  Page maps slots to offsets.

- Most common approach used in DBMSs today.
- Header keeps track of the number of used slots, the offset of the starting location of the last used slot, and a slot array, which keeps track of the location of the start of each tuple.
- To add a tuple, the slot array will grow from the beginning to the end, and the data of the tuples will grow from end to the beginning. The page is considered full when the slot array and the tuple data meet.

**Log-Structured:**  Covered in the next lecture.

![34.jpg](assets/1677726640208-baf40210-537d-41de-8fde-76fafbe40391.jpeg)

![35.jpg](assets/1677726640276-15e1624b-70aa-4dfb-b1e8-3f67326d7ef9.jpeg)

![36.jpg](assets/1677726640991-392e2c46-d1e5-4bf9-aacd-7b32c78e684c.jpeg)

![37.jpg](assets/1677726641556-1357afdc-08e0-4056-8cd3-fa3f4b9b2a38.jpeg)

![38.jpg](assets/1677726643946-fa40d113-c404-4c0c-b4b3-a616e29a6a25.jpeg)

![39.jpg](assets/1677726644514-005dde8d-3eb0-42a1-b46c-20c5a6708b21.jpeg)

![40.jpg](assets/1677726645000-3acc1749-70a0-4e2b-9662-419f0a7f033d.jpeg)

![41.jpg](assets/1677726645572-6ed159f4-aa37-4109-8c99-c532ca3ac90a.jpeg)

![42.jpg](assets/1677726646114-f063c558-ce22-44e1-aca9-82957d9030f0.jpeg)

![43.jpg](assets/1677726649133-52ceb124-724d-4feb-95e5-f50ddd26b37b.jpeg)

![44.jpg](assets/1677726649929-3a6e4893-2a05-4a34-87fa-951d33c525a0.jpeg)

![45.jpg](assets/1677726648537-8f94f40f-64eb-4666-a21b-b7b6281275d5.jpeg)

# Tuple Layout

A tuple is essentially a sequence of bytes. It is the DBMS’s job to interpret those bytes into attribute types and values.

![46.jpg](assets/1677726649134-3b4885bc-905d-408f-af48-ed12265b4297.jpeg)

**Tuple Header:**  Contains meta-data about the tuple.

- Visibility information for the DBMS’s **concurrency control** protocol (i.e., information about which transaction created/modiﬁed that tuple).
- **Bit Map** for `NULL` values.
- Note that the DBMS **does not need to store meta-data about the schema** of the database here.

![47.jpg](assets/1677726650630-0af4bffd-ae46-4863-8319-fa165547b3c9.jpeg)

**Tuple Data:**  Actual data for attributes.

- Attributes are typically stored in the order that you specify them when you create the table.
- Most DBMSs do not allow a tuple to exceed the size of a page.

**Unique Identiﬁer**:

- Each tuple in the database is assigned a unique identiﬁer.
- Most common: `page_id + (offset or slot).`
- An application cannot rely on these ids to mean anything.

![48.jpg](assets/1677726652928-9c9891c1-d1ad-4164-80aa-47093bc1b78b.jpeg)

**Denormalized Tuple Data**: If two tables are related, the DBMS can “pre-join” them, so the tables end up on the same page. This makes reads faster since the DBMS only has to load in one page rather than two separate pages. However, it makes updates more expensive since the DBMS needs more space for each tuple.

![49.jpg](assets/1677726653754-805d76bc-5624-4fc1-a2b0-0efa9d555747.jpeg)

![50.jpg](assets/1677726653735-d05c2da4-c527-40d2-9471-3c908ec151c9.jpeg)

![51.jpg](assets/1677726655931-59a64daf-362a-49a6-b342-f3ccddfe064b.jpeg)

![52.jpg](assets/1677726655266-5a3397bc-e0dc-4dbd-9b75-8c6c87265224.jpeg)

![53.jpg](assets/1677726656058-91a0b620-4c38-4324-a0f0-1b8c807cb92e.jpeg)
