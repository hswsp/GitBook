# 03 - Database Storage 1

![1.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726612688-2fb0193f-a5dd-492f-b922-a589daf5e425.jpeg#averageHue=%231d2635&clientId=u61c9f877-ad66-4&from=ui&id=u63591335&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=606409&status=done&style=none&taskId=u613023fa-82f7-4de2-bb10-b46822a6010&title=)

![2.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726610528-17b02ef3-3ddb-4b9a-9016-cd9b2c8be601.jpeg#averageHue=%23ededed&clientId=u61c9f877-ad66-4&from=ui&id=u73a124df&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=355305&status=done&style=none&taskId=ucd5cfb0e-3333-4d8c-838c-dfe79be4f1c&title=)

![3.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726609607-8ccca72f-eca2-4d42-8b97-3c3d1d0f2e6e.jpeg#averageHue=%23ededed&clientId=u61c9f877-ad66-4&from=ui&id=u298f637b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=271991&status=done&style=none&taskId=u235f4d72-a778-4b48-b2de-3e9f9e708ce&title=)

# Storage

We will focus on a “disk-oriented” DBMS architecture that assumes that the primary storage location of the database is on non-volatile disk(s).

![4.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726610275-6cdb577c-10e4-4722-ab76-d85cdfff3535.jpeg#averageHue=%23dedede&clientId=u61c9f877-ad66-4&from=ui&id=ua7411771&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=314858&status=done&style=none&taskId=u387bfec5-0706-47a3-ab98-8a67c791bf0&title=)

![5.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726609622-6576a122-b1ef-473d-af5e-6a6cdbb89385.jpeg#averageHue=%23ededed&clientId=u61c9f877-ad66-4&from=ui&id=uefc32db7&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=275211&status=done&style=none&taskId=u9ae80198-8f4c-4899-ad4b-a1349019753&title=)

At the top of the storage hierarchy, you have the devices that are closest to the CPU. This is the fastest storage, but it is also the smallest and most expensive. The further you get away from the CPU, the larger but slower the storage devices get. These devices also get cheaper per GB.

![6.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726614303-93d131ae-1874-4e0d-a491-054e95f22f4c.jpeg#averageHue=%23d9d6d3&clientId=u61c9f877-ad66-4&from=ui&id=ue9d892b2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=389648&status=done&style=none&taskId=u41d35f72-f871-40b4-baef-869e36fcba0&title=)

**Volatile Devices:**

- Volatile means that if you pull the power from the machine, then the data is lost.
- **Volatile storage supports fast random access with byte-addressable locations.**  This means that the program can jump to any byte address and get the data that is there.
- For our purposes, we will always refer to this storage class as “memory.”

**Non-Volatile Devices:**

- Non-volatile means that the storage device does not require continuous power in order for the device to retain the bits that it is storing.
- It is also **block/page addressable.**  This means that in order to read a value at a particular offset, the program ﬁrst has to load the 4 KB page into memory that holds the value the program wants to read.
- Non-volatile storage is traditionally better at sequential access (reading multiple contiguous chunks of data at the same time).
- We will refer to this as “disk.” We will not make a (major) distinction between solid-state storage (SSD) and spinning hard drives (HDD).

![7.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726614683-98ed09cc-7d8a-44e6-96a5-2d7f194b35a2.jpeg#averageHue=%23d5d0cd&clientId=u61c9f877-ad66-4&from=ui&id=u1afa5b29&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=369413&status=done&style=none&taskId=u09d1dfe5-14f9-444b-a3bf-ad137839470&title=)

There is also a relatively new class of storage devices that are becoming more popular called *persistent memory*. These devices are designed to be the best of both worlds: **almost as fast as DRAM with the persistence of disk**. We will not cover these devices in this course, and they are currently not in widespread production use. Probably the most famous example is Optane; unfortunately Intel is winding down its production as of summer 2022. Note that you may see older references to persistent memory as “non-volatile memory”.

![8.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726617088-c68d476a-b749-4c1f-9941-03adac740b6d.jpeg#averageHue=%23e7e3c4&clientId=u61c9f877-ad66-4&from=ui&id=u26700350&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=622986&status=done&style=none&taskId=u725459ed-a44d-4fa1-9a61-93e7415168d&title=)

You may see references to NVMe SSDs, where NVMe stands for non-volatile memory express. These NVMe SSDs are not the same hardware as persistent memory modules. Rather, they are typical NAND ﬂash drives that connect over an improved hardware interface. This improved hardware interface allows for much faster transfers, which leverages improvements in NAND ﬂash perfomance.

![9.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726615015-ed8837ac-b6c5-49a6-b4ba-81f880efe732.jpeg#averageHue=%23e8e7e7&clientId=u61c9f877-ad66-4&from=ui&id=ubf0f037e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=354478&status=done&style=none&taskId=u2c4c060a-7446-4346-a3d2-b8d5318b57f&title=)

Since our DBMS architecture assumes that the database is stored on disk, the components of the DBMS are responsible for ﬁguring out how to move data between non-volatile disk and volatile memory since the system cannot operate on the data directly on disk.
We will focus on hiding the latency of the disk rather than optimizations with registers and caches since getting data from disk is so slow. If reading data from the L1 cache reference took one second, reading from an SSD would take 4.4 hours, and reading from an HDD would take 3.3 weeks.

# Disk-Oriented DBMS Overview

The database is all on disk, and the data in database ﬁles is organized into pages, with the ﬁrst page being the **directory page**. To operate on the data, the DBMS needs to bring the data into memory. It does this by having a **buffer pool** that manages the data movement back and forth between disk and memory. The DBMS also has an **execution engine** that will execute queries. The execution engine will ask the buffer pool for a speciﬁc page, and the buffer pool will take care of bringing that page into memory and giving the execution engine a pointer to that page in memory. The buffer pool manager will ensure that the page is there while the execution engine operates on that part of memory.

![10.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726617332-9448b9a4-7d14-4176-84d2-c05de1ea4d86.jpeg#averageHue=%23ebebeb&clientId=u61c9f877-ad66-4&from=ui&id=uc7f5e8f1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=336181&status=done&style=none&taskId=uc244cd52-6cc9-49fc-ae61-67fa71f396f&title=)

![11.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726619249-0181a922-342b-4e07-a512-d98d34066c08.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=u0311227a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=369105&status=done&style=none&taskId=ue4ccf6d0-fd34-48ba-828f-55d31aa023a&title=)

![12.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726619982-e04573da-553a-4946-ae47-c6d2f5dd5fd1.jpeg#averageHue=%23e8e7e7&clientId=u61c9f877-ad66-4&from=ui&id=uc84fd6eb&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=426902&status=done&style=none&taskId=u3c792bdf-7f28-43b8-87b8-a6c7ae5d066&title=)

# DBMS vs. OS

A high-level design goal of the DBMS is to support databases that exceed the amount of memory available. Since reading/writing to disk is expensive, disk use must be carefully managed. We do not want large stalls from fetching something from disk to slow down everything else. We want the DBMS to be able to process other queries while it is waiting to get the data from disk.
This high-level design goal is like virtual memory, where there is a large address space and a place for the OS to bring in pages from disk.
One way to achieve this virtual memory is by using `mmap` to map the contents of a ﬁle in a process’ address space, which makes the OS responsible for moving pages back and forth between disk and memory. **Unfortunately, this means that if **​ **`**mmap**`**​ ** hits a page fault, the process will be blocked.**

- You never want to use `mmap` in your DBMS if you need to write.
- The DBMS (almost) always wants to control things itself and can do a better job at it since it knows more about the data being accessed and the queries being processed.
- The operating system is not your friend.

![13.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726620316-bbb97fec-a1cb-459a-b6b3-e7f21bde75cd.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=u97d4b634&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=412390&status=done&style=none&taskId=u3f65ca94-ab60-4a90-9bf5-4b8ad7b1766&title=)

![14.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726622459-342c9439-0560-4d2e-b3d7-fa85365ea5f9.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=u06f6cc70&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=413919&status=done&style=none&taskId=u5d807156-7142-4d47-8730-f245c9836b5&title=)

![15.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726622936-cf723fec-ee78-46a1-b579-2241d10ea9e1.jpeg#averageHue=%23eae9e9&clientId=u61c9f877-ad66-4&from=ui&id=u818eb7e2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=422922&status=done&style=none&taskId=u018960ea-da2d-4eb4-8985-030f94ec87e&title=)

![16.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726622911-5ccd102c-c851-4ddb-be5e-9081802a6336.jpeg#averageHue=%23ededed&clientId=u61c9f877-ad66-4&from=ui&id=u0044c69e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=261208&status=done&style=none&taskId=u5597a218-af82-483f-b0a9-b1fd4b02e6c&title=)

![17.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726624885-184ab3bf-99ec-412a-b8b0-71876d9d4f1f.jpeg#averageHue=%23e9e9e9&clientId=u61c9f877-ad66-4&from=ui&id=uf9034df0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=376303&status=done&style=none&taskId=u7c7b2040-78ee-4271-8b09-5fad42eb272&title=)

TLB ( Translation Lookaside Buffer。MMU 为了加速查找页表，使用的 cache。用来加速 MMU 的转化速度)是从虚拟内存地址到物理内存地址转换的高速缓存。当处理器更改地址的虚拟到物理映射时，它需要告诉其他处理器在其缓存中使该映射无效。这个过程被称为"[TLB shootdowns](https://juejin.cn/post/6844904084957315086)"。
It is possible to use the OS by using:

- `madvise`: Tells the OS know when you are planning on reading certain pages.
- `mlock`: Tells the OS to not swap memory ranges out to disk.
- `msync`: Tells the OS to ﬂush memory ranges out to disk.

![18.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726625569-e08aff22-d5d3-41ba-9d3e-582730ba316f.jpeg#averageHue=%23ecebeb&clientId=u61c9f877-ad66-4&from=ui&id=u67ee284b&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=438194&status=done&style=none&taskId=u5ef1088f-f5d9-42d7-af8f-7c6794bd966&title=)

![19.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726626605-a4a85d7d-f028-4611-a876-6563a55b7cef.jpeg#averageHue=%23ededed&clientId=u61c9f877-ad66-4&from=ui&id=ua18e41f0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=290537&status=done&style=none&taskId=u4ea110b3-5e74-43b6-bc31-ff6c8ea7b76&title=)

We do not advise using `mmap` in a DBMS for correctness and performance reasons.
Even though the system will have functionalities that seem like something the OS can provide, having the DBMS implement these procedures itself gives it better control and performance.

![20.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726629966-08d46fb2-18eb-4734-8ae7-142918b691ec.jpeg#averageHue=%23ebeaea&clientId=u61c9f877-ad66-4&from=ui&id=uc6253275&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=673411&status=done&style=none&taskId=u9e33358b-4f03-4590-b9f8-3140a9adc4d&title=)

![21.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726627087-93c5a955-9be5-4fdd-a07c-6158b17db56a.jpeg#averageHue=%23ededed&clientId=u61c9f877-ad66-4&from=ui&id=u29cd1574&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=287854&status=done&style=none&taskId=u88cd3b93-103d-4b59-b585-287b92f5922&title=)

![22.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726627651-28e94baf-2ca6-495a-82c8-709d4d98cbaa.jpeg#averageHue=%23f0f0f0&clientId=u61c9f877-ad66-4&from=ui&id=uac81daa5&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=163086&status=done&style=none&taskId=u5ac7b368-69df-4a82-be5e-fa755b2cdc9&title=)

# File Storage

In its most basic form, a DBMS stores a database as ﬁles on disk. Some may use a ﬁle hierarchy, others may use a single ﬁle (e.g., SQLite).
The OS does not know anything about the contents of these ﬁles. Only the DBMS knows how to decipher their contents, since it is encoded in a way speciﬁc to the DBMS.

![23.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726629502-b80231c6-d446-4d9b-8a31-6984b454df05.jpeg#averageHue=%23ececec&clientId=u61c9f877-ad66-4&from=ui&id=u775620db&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=319264&status=done&style=none&taskId=ua0fc5fc0-e5c1-45f6-9be3-2f8ac323c10&title=)

The DBMS’s *storage manager* is responsible for managing a database’s ﬁles. It represents the ﬁles **as a collection of pages**. It also keeps track of what data has been read and written to pages as well how much free space there is in these pages.

![24.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726630551-ac052419-c475-4ec5-ac97-adf3cdadf473.jpeg#averageHue=%23ececec&clientId=u61c9f877-ad66-4&from=ui&id=u19134231&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=313417&status=done&style=none&taskId=u0404dea2-bb11-42ff-b334-ef5b237fb40&title=)

# Database Pages

The DBMS organizes the database across one or more ﬁles in ﬁxed-size blocks of data called pages. Pages can contain different kinds of data (tuples, indexes, etc). Most systems will not mix these types within pages. Some systems will require that pages are **self-contained**, meaning that all the information needed to read each page is on the page itself.
Each page is given a unique identiﬁer. If the database is a single ﬁle, then the page id can just be the ﬁle offset. Most DBMSs have an indirection layer that **maps a page id to a ﬁle path and offset**. The upper levels of the system will ask for a speciﬁc page number. Then, the storage manager will have to turn that page number into a ﬁle and an offset to ﬁnd the page.

![25.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726631159-2f0533ed-941e-4ecf-9a7c-cdcb179a3af3.jpeg#averageHue=%23ededed&clientId=u61c9f877-ad66-4&from=ui&id=u94d0b21d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=303582&status=done&style=none&taskId=u83188305-4a9a-4f91-87b4-1ba317c3217&title=)

Most DBMSs uses ﬁxed-size pages to avoid the engineering overhead needed to support variable-sized pages. For example, with variable-size pages, deleting a page could create a hole in ﬁles that the DBMS cannot easily ﬁll with new pages.
There are three concepts of pages in DBMS:

1. Hardware page (usually 4 KB).
2. OS page (4 KB).
3. Database page (1-16 KB).

**The storage device guarantees an atomic write of the size of the hardware page.**  If the hardware page is 4 KB and the system tries to write 4 KB to the disk, either all 4 KB will be written, or none of it will. This means that if our database page is larger than our hardware page, the DBMS will have to take extra measures to ensure that the data gets written out safely since the program can get partway through writing a database page to disk when the system crashes.

![26.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726632283-37ebae70-9b82-4768-bdab-929798a49595.jpeg#averageHue=%23eae9e9&clientId=u61c9f877-ad66-4&from=ui&id=uf204345d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=443106&status=done&style=none&taskId=ud823e6ca-e97d-473d-b49b-21594894dde&title=)

# Database Heap

There are a couple of ways to ﬁnd the location of the page a DBMS wants on the disk, and **heap ﬁle** organization is one of those ways. A heap ﬁle is an unordered collection of pages where tuples are stored in random order.

![27.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726634356-1ad79dc0-f024-4318-b645-05523ed977a5.jpeg#averageHue=%23ebebeb&clientId=u61c9f877-ad66-4&from=ui&id=u49d0320e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=338723&status=done&style=none&taskId=u1265033d-a975-4fdd-ada9-5b2bf2bc054&title=)

The DBMS can locate a page on disk given a page id by using a **linked list** of pages or a page directory.

1. ** Linked List**: Header page holds pointers to a list of free pages and a list of data pages. However, if the DBMS is looking for a speciﬁc page, it has to do a sequential scan on the data page list until it ﬁnds the page it is looking for.
2. **Page Directory**: DBMS maintains special pages that track locations of data pages along with the amount of free space on each page.

![28.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726635474-43c3f100-78c9-4b27-bd9e-07c6bdd77f9a.jpeg#averageHue=%23ececec&clientId=u61c9f877-ad66-4&from=ui&id=uc31c4227&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=351721&status=done&style=none&taskId=u74ef2d1d-579a-4d8d-9e21-aa789bcc9db&title=)

![29.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726635965-5f429e6c-19fd-4b1b-b6c2-de8acdb1f08a.jpeg#averageHue=%23ececec&clientId=u61c9f877-ad66-4&from=ui&id=u23986c4e&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=367498&status=done&style=none&taskId=u3cb0539c-f609-4f2e-886d-6940947f490&title=)

![30.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726635960-6c76bf0e-5fff-45e9-a7fd-007ea1e4ca6f.jpeg#averageHue=%23ebebeb&clientId=u61c9f877-ad66-4&from=ui&id=ufad37864&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=350036&status=done&style=none&taskId=u82b85d56-81dc-4ed3-9951-d1f73073f99&title=)

![31.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726637542-8f0a72cd-c407-4466-b224-f2c34e17d94a.jpeg#averageHue=%23ebeaea&clientId=u61c9f877-ad66-4&from=ui&id=u8c584821&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=399955&status=done&style=none&taskId=u80c37e7a-cc23-4630-b8f7-b841fe5c4fb&title=)

![32.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726637055-8de92311-17e3-480f-830c-48605a90f9d6.jpeg#averageHue=%23f0f0f0&clientId=u61c9f877-ad66-4&from=ui&id=u7ceccc92&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=159768&status=done&style=none&taskId=u72f6d59b-6ac1-4cfb-aed5-a80a5fbd487&title=)

# Page Layout

Every page includes a header that records meta-data about the page’s contents:

- Page size.
- Checksum.
- DBMS version.
- Transaction visibility.
- Self-containment. (Some systems like Oracle require this.)

![33.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726639646-2f882de5-7c22-457c-920c-f333efaec637.jpeg#averageHue=%23ececec&clientId=u61c9f877-ad66-4&from=ui&id=ub4e89aa9&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=305852&status=done&style=none&taskId=u5741eb21-975a-4672-b1ea-dfbcdec1392&title=)

A **strawman approach** to laying out data is to keep track of how many tuples the DBMS has stored in a page and then append to the end every time a new tuple is added. However, ***problems arise when tuples are deleted or when tuples have variable-length attributes***.
There are two **main approaches** to laying out data in pages: (1) **slotted-pages** and (2) **log-structured**.
**Slotted Pages:**  Page maps slots to offsets.

- Most common approach used in DBMSs today.
- Header keeps track of the number of used slots, the offset of the starting location of the last used slot, and a slot array, which keeps track of the location of the start of each tuple.
- To add a tuple, the slot array will grow from the beginning to the end, and the data of the tuples will grow from end to the beginning. The page is considered full when the slot array and the tuple data meet.

**Log-Structured:**  Covered in the next lecture.

![34.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726640208-baf40210-537d-41de-8fde-76fafbe40391.jpeg#averageHue=%23eeeeee&clientId=u61c9f877-ad66-4&from=ui&id=ua912a38d&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=277491&status=done&style=none&taskId=ue35edb9f-f219-4f5b-83a1-16f9fcf57ea&title=)

![35.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726640276-15e1624b-70aa-4dfb-b1e8-3f67326d7ef9.jpeg#averageHue=%23ececec&clientId=u61c9f877-ad66-4&from=ui&id=u8d783c25&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=312833&status=done&style=none&taskId=uc38d299e-d469-48e7-8a81-7b9c7a16147&title=)

![36.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726640991-392e2c46-d1e5-4bf9-aacd-7b32c78e684c.jpeg#averageHue=%23ececec&clientId=u61c9f877-ad66-4&from=ui&id=uee2918b3&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=308165&status=done&style=none&taskId=ucf8a5e41-dbd9-40e2-aa90-8d88558786f&title=)

![37.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726641556-1357afdc-08e0-4056-8cd3-fa3f4b9b2a38.jpeg#averageHue=%23ececec&clientId=u61c9f877-ad66-4&from=ui&id=u3b4393b4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=313137&status=done&style=none&taskId=ufb851f9c-420c-4435-b1d6-e9d3483ef1b&title=)

![38.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726643946-fa40d113-c404-4c0c-b4b3-a616e29a6a25.jpeg#averageHue=%23ebebeb&clientId=u61c9f877-ad66-4&from=ui&id=u38692a74&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=350527&status=done&style=none&taskId=uf56bf661-3630-4e53-8d1e-15a5ee34fa2&title=)

![39.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726644514-005dde8d-3eb0-42a1-b46c-20c5a6708b21.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=u11325787&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=384236&status=done&style=none&taskId=uea4013a3-3d34-4411-9ed4-357f7a178a3&title=)

![40.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726645000-3acc1749-70a0-4e2b-9662-419f0a7f033d.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=uaa8cae57&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=369655&status=done&style=none&taskId=u7887366b-024c-4939-acf7-48887d1b71a&title=)

![41.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726645572-6ed159f4-aa37-4109-8c99-c532ca3ac90a.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=u3e20620a&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=370633&status=done&style=none&taskId=u8bd39d06-bc4f-4121-a573-01de0871ce2&title=)

![42.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726646114-f063c558-ce22-44e1-aca9-82957d9030f0.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=uc88bf90f&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=365826&status=done&style=none&taskId=udbe6c012-cc27-43eb-bfe9-8aa1c039c80&title=)

![43.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726649133-52ceb124-724d-4feb-95e5-f50ddd26b37b.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=u250dd6b2&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=364087&status=done&style=none&taskId=u0c835007-616d-4c61-a1a3-290587ada38&title=)

![44.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726649929-3a6e4893-2a05-4a34-87fa-951d33c525a0.jpeg#averageHue=%23edebeb&clientId=u61c9f877-ad66-4&from=ui&id=u3ecd7bb0&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=398508&status=done&style=none&taskId=u8ec4abe2-5e6f-49be-9ff5-823850d10e1&title=)

![45.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726648537-8f94f40f-64eb-4666-a21b-b7b6281275d5.jpeg#averageHue=%23f0f0f0&clientId=u61c9f877-ad66-4&from=ui&id=u57fd92f4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=156359&status=done&style=none&taskId=u92798d43-127a-42de-9e31-f33d7eef326&title=)

# Tuple Layout

A tuple is essentially a sequence of bytes. It is the DBMS’s job to interpret those bytes into attribute types and values.

![46.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726649134-3b4885bc-905d-408f-af48-ed12265b4297.jpeg#averageHue=%23efefef&clientId=u61c9f877-ad66-4&from=ui&id=u7a004718&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=211241&status=done&style=none&taskId=u1dd280c5-d5bd-4320-afd0-134940e3b93&title=)

**Tuple Header:**  Contains meta-data about the tuple.

- Visibility information for the DBMS’s **concurrency control** protocol (i.e., information about which transaction created/modiﬁed that tuple).
- **Bit Map** for `NULL` values.
- Note that the DBMS **does not need to store meta-data about the schema** of the database here.

![47.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726650630-0af4bffd-ae46-4863-8319-fa165547b3c9.jpeg#averageHue=%23ededed&clientId=u61c9f877-ad66-4&from=ui&id=u213dd494&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=280993&status=done&style=none&taskId=u9e930e34-a7d5-419f-8958-19f8bb823f0&title=)

**Tuple Data:**  Actual data for attributes.

- Attributes are typically stored in the order that you specify them when you create the table.
- Most DBMSs do not allow a tuple to exceed the size of a page.

**Unique Identiﬁer**:

- Each tuple in the database is assigned a unique identiﬁer.
- Most common: `page_id + (offset or slot).`
- An application cannot rely on these ids to mean anything.

![48.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726652928-9c9891c1-d1ad-4164-80aa-47093bc1b78b.jpeg#averageHue=%23ebebeb&clientId=u61c9f877-ad66-4&from=ui&id=ud72a16ce&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=366637&status=done&style=none&taskId=u64174fb2-b694-462c-865d-209cf4b1385&title=)

**Denormalized Tuple Data**: If two tables are related, the DBMS can “pre-join” them, so the tables end up on the same page. This makes reads faster since the DBMS only has to load in one page rather than two separate pages. However, it makes updates more expensive since the DBMS needs more space for each tuple.

![49.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726653754-805d76bc-5624-4fc1-a2b0-0efa9d555747.jpeg#averageHue=%23eaeaea&clientId=u61c9f877-ad66-4&from=ui&id=ue818b7a4&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=378645&status=done&style=none&taskId=ub11c3997-b2ce-4ea0-9925-c47de996d5e&title=)

![50.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726653735-d05c2da4-c527-40d2-9471-3c908ec151c9.jpeg#averageHue=%23ecebeb&clientId=u61c9f877-ad66-4&from=ui&id=uacd1d1ad&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=336482&status=done&style=none&taskId=uafb6aa92-401a-4143-809e-748a7cf5939&title=)

![51.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726655931-59a64daf-362a-49a6-b342-f3ccddfe064b.jpeg#averageHue=%23e8e7e7&clientId=u61c9f877-ad66-4&from=ui&id=u7c2826f1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=494691&status=done&style=none&taskId=u961f96a2-c330-44d3-9902-3c9f67ef390&title=)

![52.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726655266-5a3397bc-e0dc-4dbd-9b75-8c6c87265224.jpeg#averageHue=%23efefef&clientId=u61c9f877-ad66-4&from=ui&id=ufb414aa1&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=217000&status=done&style=none&taskId=ua91ec0f2-ea3e-4655-baa4-9fbdaa486df&title=)

![53.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/22382307/1677726656058-91a0b620-4c38-4324-a0f0-1b8c807cb92e.jpeg#averageHue=%23f0f0f0&clientId=u61c9f877-ad66-4&from=ui&id=ueccf2041&originHeight=1688&originWidth=3000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=170853&status=done&style=none&taskId=u89d62a46-c5a5-4268-8033-b4c535ed975&title=)
