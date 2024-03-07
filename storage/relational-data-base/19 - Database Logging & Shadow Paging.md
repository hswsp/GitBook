# 19 - Database Logging & Shadow Paging

![1.jpg](assets/1-20231123131906-znoie1v.jpg)

![2.jpg](assets/2-20231123131906-8znbrqf.jpg)

![3.jpg](assets/3-20231123131906-vhv366l.jpg)

# Crash Recovery

*Recovery algorithms* are techniques to ensure database consistency, transaction atomicity, and durability despite failures. When a crash occurs, all the data in memory that has not been committed to disk is at risk of being lost. Recovery algorithms act to prevent loss of information after a crash.

![4.jpg](assets/4-20231123131906-uoczq6s.jpg)

![5.jpg](assets/5-20231123131906-5h0s67v.jpg)

![6.jpg](assets/6-20231123131906-1sfqq1j.jpg)

![7.jpg](assets/7-20231123131906-mn17nxi.jpg)

Every recovery algorithm has two parts:

• Actions during normal transaction processing to ensure that the DBMS can recover from a failure.

• Actions after a failure to recover the database to a state that ensures atomicity, consistency, and durability.

The key primitives that used in recovery algorithms are UNDO and REDO. Not all algorithms use both primitives.

• **UNDO**: The process of removing the effects of an incomplete or aborted transaction.

• **REDO**: The process of re-applying the effects of a committed transaction for durability.

![8.jpg](assets/8-20231123131906-spii5gw.jpg)

![9.jpg](assets/9-20231123131906-rn2lj8h.jpg)

![10.jpg](assets/10-20231123131906-kpqoak8.jpg)

# Storage Types

• **Volatile Storage**

– Data does not persist after power is lost or program exits.

– Examples: DRAM, SRAM,.

• **Non-Volatile Storage**

– Data persists after losing power or program exists.

– Examples: HDD, SDD.

• **Stable Storage**

– A non-existent form of non-volatile storage that survives all possible failures scenarios.

– Use multiple storage devices to approximate.

![11.jpg](assets/11-20231123131906-5awe9nc.jpg)

# Failure Classification

Because the DBMS is divided into different components based on the underlying storage device, there are a number of different types of failures that the DBMS needs to handle. Some of these failures are recoverable while others are not.

![12.jpg](assets/12-20231123131906-h8i4uj3.jpg)

## Type #1:# Transaction Failures

*Transactions failures* occur when a transaction reaches an error and must be aborted. Two types of errors that can cause transaction failures are logical errors and internal state errors.

• **Logical Errors**: A transaction cannot complete due to some internal error condition (e.g., integrity, constraint violation).

• **Internal State Errors**: The DBMS must terminate an active transaction due to an error condition (e.g., deadlock)

![13.jpg](assets/13-20231123131906-dhk2qka.jpg)

## Type #2:# System Failures

*System failures* are unintented failures in the underlying software or hardware that hosts the DBMS. These failures must be accounted for in crash recovery protocols.

• **Software Failure**: There is a problem with the DBMS implementation (e.g., uncaught divide-by-zero exception) and the system has to halt.

• **Hardware Failure**: The computer hosting the DBMS crashes (e.g., power plug gets pulled). We assume that non-volatile storage contents are not corrupted by system crash. This is called the ”Failstop” assumption and simplifies the process recovery.

![14.jpg](assets/14-20231123131906-z77ullw.jpg)

## Type #3:# Storage Media Failure

*Storage media failures* are non-repairable failures that occur when the physical storage device is damaged. When the storage media fails, the DBMS must be restored from an archived version. The DBMS cannot recover from a storage failure and requires human intervention.

• **Non-Repairable Hardware Failure**: A head crash or similar disk failure destroys all or parts of non-volatile storage. Destruction is assumed to be detectable.

![15.jpg](assets/15-20231123131906-xl1geuy.jpg)

![16.jpg](assets/16-20231123131906-0tfb6te.jpg)

![17.jpg](assets/17-20231123131906-sk7ku87.jpg)

# Buffer Pool Management Policies

The DBMS needs to ensure the following guarantees:

• The changes for any transaction are durable once the DBMS has told somebody that it committed.

• No partial changes are durable if the transaction aborted.

![18.jpg](assets/18-20231123131906-wjyrys7.jpg)

![19.jpg](assets/19-20231123131906-pwjyruj.jpg)

![20.jpg](assets/20-20231123131906-h56ekbn.jpg)

![21.jpg](assets/21-20231123131906-iuo8m6r.jpg)

![22.jpg](assets/22-20231123131906-abiq2q6.jpg)

![23.jpg](assets/23-20231123131906-dnx64t4.jpg)

![24.jpg](assets/24-20231123131906-lv2rumt.jpg)

A *steal policy* dictates whether the DBMS allows an uncommitted transaction to overwrite the most recent committed value of an object in **non**-volatile storage (can a transaction write uncommitted changes belonging to a different transaction to disk?).

• **STEAL**: Is allowed

• **NO-STEAL**: Is not allowed.

![25.jpg](assets/25-20231123131906-i8s4p2s.jpg)

A *force policy* dictates whether the DBMS requires that all updates made by a transaction are reflected on **non**-volatile storage before the transaction is allowed to commit (ie. return a commit message back to the client).

• **FORCE**: Is required

• **NO-FORCE**: Is not required

Force writes make it easier to recover since all of the changes are preserved but result in poor runtime performance.

The **easiest** buffer pool management policy to implement is called **NO-STEAL + FORCE**. In this policy, the DBMS never has to undo changes of an aborted transaction because the changes were not written to disk. It also never has to redo changes of a committed transaction because all the changes are guaranteed to be written to disk at commit time. An example of **NO-STEAL + FORCE** is show in Figure 1.

![26.jpg](assets/26-20231123131906-neyk2ni.jpg)

![27.jpg](assets/27-20231123131906-p2532w2.jpg)

![28.jpg](assets/28-20231123131906-xx6u89c.jpg)

![29.jpg](assets/29-20231123131906-to6e37s.jpg)

![30.jpg](assets/30-20231123131906-arbedt0.jpg)

![31.jpg](assets/31-20231123131906-ppz3n2m.jpg)

![32.jpg](assets/32-20231123131906-tmbb9he.jpg)

**Figure 1: DBMS using NO-STEAL + FORCE Example** – All changes from a transaction are only written to disk when the transaction is committed. Once the schedule begins at Step #1,# changes from $T_1$ and $T_2$ are written to the buffer pool. Because of the FORCE policy, when $T_2$ commits at Step #2,# all of its changes must be written to disk. To do this, the DBMS makes a copy of the memory in disk, applies only the changes from $T_2$ , and writes it back to disk. This is because NO-STEAL prevents the uncommitted changes from $T_1$ to be written to disk. At Step #3,# it is trivial for the DBMS to rollback $T_1$ since no dirty changes from $T_1$ are on disk.

![33.jpg](assets/33-20231123131906-scsea9w.jpg)

A limitation of **NO STEAL + FORCE** is that all of the data (ie. the write set) that a transaction needs to modify must fit into memory. Otherwise, that transaction cannot execute because the DBMS is not allowed to write out dirty pages to disk before the transaction commits.

![34.jpg](assets/34-20231123131906-wnlx0k3.jpg)

# Shadow Paging

Shadow Paging is an improvement upon the previous scheme where the DBMS copies pages on write to maintain two separate versions of the database:

• *master*: Contains only changes from committed txns.

• *shadow*: Temporary database with changes made from uncommitted transactions.

![35.jpg](assets/35-20231123131906-5vj0a3i.jpg)

Updates are only made in the shadow copy. When a transaction commits, the shadow copy is atomically switched to become the new master. The old master is eventually garbage collected. This is an example of a **NO-STEAL + FORCE** system. A high-level example of shadow paging is shown in Figure 2.

![36.jpg](assets/36-20231123131906-brsvk13.jpg)

![37.jpg](assets/37-20231123131906-0hlhebm.jpg)

![38.jpg](assets/38-20231123131906-r671o67.jpg)

![39.jpg](assets/39-20231123131906-nqtmg9z.jpg)

![40.jpg](assets/40-20231123131906-j7sj33a.jpg)

![41.jpg](assets/41-20231123131906-bt3e0rb.jpg)

![42.jpg](assets/42-20231123131906-x8miwj4.jpg)

![43.jpg](assets/43-20231123131906-l8oxdra.jpg)

**Figure 2: Shadow Paging** – The database root points to a master page table which in turn points to the pages on disk (all of which contain committed data). When an updating transaction occurs, a shadow page table is created that points to the same pages as the master. **Modifications are made to a temporary space on disk** and the shadow table is updated. To complete the commit, the database root pointer is redirected to the shadow table, which becomes the new master.

![44.jpg](assets/44-20231123131906-2s6nuj5.jpg)

## Recovery

• **Undo**: Remove the shadow pages. Leave the master and DB root pointer alone.

• **Redo**: Not needed at all.

![45.jpg](assets/45-20231123131906-70oa0je.jpg)

## Disadvantages

A disadvantage of shadow paging is that copying the entire page table is expensive. In reality, only paths in the tree that lead to updated leaf nodes need to be copied, not the entire tree. In addition, the commit overhead of shadow paging is high. Commits require the page table, and root, in addition to every updated page to be flushed. This causes fragmented data and also requires garbage collection. Another issue is that this only supports one writer transaction at a time or transactions in a batch.

![46.jpg](assets/46-20231123131906-b07b5yp.jpg)

# Journal File

When a transaction modifies a page, the DBMS copies the original page to a separate journal file before overwriting the master version. After restarting, if a journal file exists, then the DBMS restores it to undo changes from uncommited transactions.

![47.jpg](assets/47-20231123131906-c1hjt9z.jpg)

![48.jpg](assets/48-20231123131906-jogmg26.jpg)

![49.jpg](assets/49-20231123131906-hw3h0qk.jpg)

![50.jpg](assets/50-20231123131906-vfxvvtm.jpg)

![51.jpg](assets/51-20231123131906-ps76l4n.jpg)

![52.jpg](assets/52-20231123131906-ndmm7yy.jpg)

![53.jpg](assets/53-20231123131906-le0hgfb.jpg)

![54.jpg](assets/54-20231123131906-4ub8yqc.jpg)

![55.jpg](assets/55-20231123131906-oy1pmkh.jpg)

![56.jpg](assets/56-20231123131906-tp52g87.jpg)

# Write-Ahead Logging

With write-ahead logging, the DBMS records all the changes made to the database in a log file (on stable storage) before the change is made to a disk page. The log contains sufficient information to perform the necessary undo and redo actions to restore the database after a crash. The DBMS must write to disk the log file records that correspond to changes made to a database object before it can flush that object to disk. An example of WAL is shown in Figure 3. **WAL is an example of a STEAL + NO-FORCE system**.

![57.jpg](assets/57-20231123131906-ff5j2rw.jpg)

In shadow paging, the DBMS was required to perform writes to random non-contiguous pages on disk. Write-ahead logging allows the DBMS to convert random writes into sequential writes to optimize performance. Thus, almost every DBMS uses write-ahead logging (WAL) because it has the fastest runtime performance. But the DBMS’s recovery time with WAL is slower than shadow paging because it has to **replay the log.**

![58.jpg](assets/58-20231123131906-w3i2h0q.jpg)

## Implementation

The DBMS first stages all of a transaction’s log records in volatile storage. All log records pertaining to an updated page are then written to non-volatile storage before the page itself is allowed to be overwritten in non-volatile storage. A transaction is not considered committed until all its log records have been written to stable storage.

When the transaction starts, write a `<BEGIN>` record to the log for each transaction to mark its starting point.

When a transaction finishes, write a `<COMMIT>` record to the log and make sure all log records are flushed before it returns an acknowledgment to the application.

Each log entry contains information necessary to rewind or replay the changes to a single object:

• Transaction ID.

• Object ID.

• Before Value (used for UNDO).

• After Value (used for REDO).

![59.jpg](assets/59-20231123131906-ye6n6dh.jpg)

![60.jpg](assets/60-20231123131906-nydx188.jpg)

![61.jpg](assets/61-20231123131906-sln1727.jpg)

![62.jpg](assets/62-20231123131906-pnpi9d6.jpg)

![63.jpg](assets/63-20231123131906-2xdce3w.jpg)

![64.jpg](assets/64-20231123131906-2b9uqgr.jpg)

![65.jpg](assets/65-20231123131906-pc0bdvz.jpg)

![66.jpg](assets/66-20231123131906-weorb4z.jpg)

The DBMS must flush all of a transaction’s log entries to disk **before** it can tell the outside world that a transaction has successfully committed. The system can use the “group commit” optimization to batch multiple log flushes together to amortize overhead. Flushes happen either when the log buffer is full, or if sufficient time has passed between successive flushes. The DBMS can write dirty pages to disk whenever it wants to, as long as it is after flushing the corresponding log records.

![67.jpg](assets/67-20231123131906-cjlzh1u.jpg)

![68.jpg](assets/68-20231123131906-ve1ut0v.jpg)

![69.jpg](assets/69-20231123131906-h1pcjy4.jpg)

![70.jpg](assets/70-20231123131906-mzrjthq.jpg)

![71.jpg](assets/71-20231123131906-oy66hr1.jpg)

![72.jpg](assets/72-20231123131906-d21hm8r.jpg)

![73.jpg](assets/73-20231123131906-7ga1i6h.jpg)

![74.jpg](assets/74-20231123131906-ibkrhu8.jpg)

![75.jpg](assets/75-20231123131906-45gud5c.jpg)

![76.jpg](assets/76-20231123131906-6j9hjj8.jpg)

## Log-Structured Systems

In a log-structured DBMS, log records of transactions are written to an in-memory buffer called the **MemTable**. When this buffer is full, it is flushed to disk. This approach **still requires a distinct write-ahead log**. This is due to the fact that **flushes of the WAL are typically more frequent than flushes of the MemTable**, and the WAL may contain uncommitted transactions. **The WAL is used to recreate the in-memory MemTable while recovering from a crash.**

![77.jpg](assets/77-20231123131906-zyvm1x0.jpg)

![78.jpg](assets/78-20231123131906-90inf8p.jpg)

# Logging Schemes

The contents of a log record can vary based on the implementation.

**Physical Logging:**

• Record the byte-level changes made to a specific location in the database.

• Example: git diff

**Logical Logging:**

- Record the high level operations executed by transactions.
- Not necessarily restricted to a single page.
- Requires less data written in each log record than physical logging because each record can update multiple tuples over multiple pages. However, it is difficult to implement recovery with logical logging when there are concurrent transactions in a non-deterministic concurrency control scheme. Additionally recovery takes longer because you must re-execute every transaction.
- Example: The `UPDATE`, `DELETE`, and `INSERT` queries invoked by a transaction.

**Physiological Logging:**

• Hybrid approach where **log records target a single page** but does not specify data organization of the page. That is, identify tuples based on a slot number in the page without specifying exactly where in the page the change is located. Therefore the DBMS can reorganize pages after a log record has been written to disk.

• Most common approach used in DBMSs.

![79.jpg](assets/79-20231123131906-2zbf0p7.jpg)

![80.jpg](assets/80-20231123131906-fajuhu2.jpg)

![81.jpg](assets/81-20231123131906-xk7ggu6.jpg)

![82.jpg](assets/82-20231123131906-nd0w5qt.jpg)

# Checkpoints

The main problem with a WAL-based DBMS is that the log file will grow forever. After a crash, the DBMS has to replay the entire log, which can take a long time if the log file is large. Thus, the DBMS can periodically take a checkpoint where it flushes all buffers out to disk.

![83.jpg](assets/83-20231123131906-m5lsnme.jpg)

How often the DBMS should take a checkpoint depends on the application’s performance and downtime requirements. Taking a checkpoint too often causes the DBMS’s runtime performance to degrade. But waiting a long time between checkpoints can potentially be just as bad, as the system’s recovery time after a restart increases.

## Blocking Checkpoint Implementation:

• The DBMS stops accepting new transactions and waits for all active transactions to complete.

• Flush all log records and dirty blocks currently residing in main memory to stable storage.

• Write a <`CHECKPOINT`> entry to the log and flush to stable storage.

![84.jpg](assets/84-20231123131906-y3p4buw.jpg)

![85.jpg](assets/85-20231123131906-ro5cz7f.jpg)

![86.jpg](assets/86-20231123131906-nrciavq.jpg)

![87.jpg](assets/87-20231123131906-cnnhurk.jpg)

![88.jpg](assets/88-20231123131906-bk8814q.jpg)

![89.jpg](assets/89-20231123131906-9xub0fa.jpg)

![90.jpg](assets/90-20231123131906-b2jjc42.jpg)

![91.jpg](assets/91-20231123131906-q3iz5tl.jpg)

![92.jpg](assets/92-20231123131906-rdxu2mt.jpg)
