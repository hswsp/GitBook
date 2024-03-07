# 16 - Two-Phase Locking in Databases

![1.jpg](assets/1-20231123131905-gzbwthg.jpg)

![2.jpg](assets/2-20231123131905-02451l3.jpg)

![3.jpg](assets/3-20231123131905-d0wbiw3.jpg)

![4.jpg](assets/4-20231123131905-dd0cl3n.jpg)

![5.jpg](assets/5-20231123131905-et6nvfg.jpg)

# Transaction Locks

A DBMS uses *locks* to dynamically generate an execution schedule for transactions that is serializable without knowing each transaction’s read/write set ahead of time. These locks protect database objects during concurrent access when there are multiple readers and writes. The DBMS contains a centralized *lock manager* that decides whether a transaction can acquire a lock or not. It also provides a global view of whats going on inside the system.

![6.jpg](assets/6-20231123131905-yigcl24.jpg)

![7.jpg](assets/7-20231123131905-r5y85ja.jpg)

There are two basic types of locks:

- **Shared Lock (S-LOCK):**  A shared lock that allows multiple transactions to read the same object at the same time. If one transaction holds a shared lock, then another transaction can also acquire that same shared lock.
- **Exclusive Lock (X-LOCK)** : An exclusive lock allows a transaction to modify an object. This lock prevents other transactions from taking any other lock (`S-LOCK` or `X-LOCK`) on the object. Only one transaction can hold an exclusive lock at a time.

Transactions must request locks (or upgrades) from the lock manager. The lock manager grants or blocks requests based on what locks are currently held by other transactions. Transactions must release locks when they no longer need them to free up the object. The lock manager updates its internal lock-table with information about which transactions hold which locks and which transactions are waiting to acquire locks.

![8.jpg](assets/8-20231123131905-7y6uvqy.jpg)

![9.jpg](assets/9-20231123131905-0bnvg1w.jpg)

![10.jpg](assets/10-20231123131905-ee35mce.jpg)

The DBMS’s lock-table does not need to be durable since any transaction that is active (i.e., still running) when the DBMS crashes is automatically aborted.

Just the usage of locks does not automatically resolve all the issues associated with concurrent transactions. Locks need to be complemented by a concurrency control protocol.

![11.jpg](assets/11-20231123131905-tk71s0m.jpg)

![12.jpg](assets/12-20231123131905-z5l8cif.jpg)

# Two-Phase Locking

Two-Phase locking (2PL) is a pessimistic concurrency control protocol that uses locks to determine whether **a transaction** is allowed to access an object in the database on the ﬂy. The protocol does not need to know all of the queries that a transaction will execute ahead of time.

**Phase #1– Growing**: In the growing phase, each transaction requests the locks that it needs from the DBMS’s lock manager. The lock manager grants/denies these lock requests.

**Phase #2– Shrinking**: Transactions enter the shrinking phase immediately after it releases its ﬁrst lock. In the shrinking phase, transactions are only allowed to release locks. They are not allowed to acquire new ones.

![13.jpg](assets/13-20231123131905-8p8f0e7.jpg)

On its own, 2PL is sufﬁcient to guarantee **conﬂict serializability**. It generates schedules whose precedence graph is acyclic. But it is susceptible to *cascading aborts*, which is when a transaction aborts and now another transaction must be rolled back, which results in wasted work.

![14.jpg](assets/14-20231123131905-ibaajvi.jpg)

![15.jpg](assets/15-20231123131905-3wccouf.jpg)

![16.jpg](assets/16-20231123131905-k7csisc.jpg)

![17.jpg](assets/17-20231123131905-sohub2y.jpg)

![18.jpg](assets/18-20231123131905-smkjigz.jpg)

![19.jpg](assets/19-20231123131905-gmvwkze.jpg)

2PL can still have dirty reads and it can also lead to deadlocks. There are also potential schedules that are serializable but would not be allowed by 2PL (locking can limit concurrency).

![20.jpg](assets/20-20231123131905-0rw2rdu.jpg)

![21.jpg](assets/21-20231123131905-x7tz1s9.jpg)

## Strong Strict Two-Phase Locking

A schedule is *strict* if any value written by a transaction is never read or overwritten by another transaction until the ﬁrst transaction commits. *Strong Strict 2PL* (also known as ***Rigorous***​ ** 2PL**) is a variant of 2PL where the transactions only release locks when they commit.

The advantage of this approach is that the DBMS does not incur *cascading aborts*. The DBMS can also reverse the changes of an aborted transaction by restoring the original values of modiﬁed tuples. However, Strict 2PL generates more cautious/pessimistic schedules that **limit concurrency.**

![22.jpg](assets/22-20231123131905-qkjexvk.jpg)

![23.jpg](assets/23-20231123131905-o5rdhce.jpg)

![24.jpg](assets/24-20231123131905-iwn41bo.jpg)

![25.jpg](assets/25-20231123131906-fervzyu.jpg)

![26.jpg](assets/26-20231123131906-gdzfijz.jpg)

![27.jpg](assets/27-20231123131906-pvtc8v5.jpg)

## Universe of Schedules

$SerialSchedules ⊂ StrongStrict2PL ⊂ ConflictSerializableSchedules ⊂ V iewSerializableSchedules ⊂ AllSchedules$

![28.jpg](assets/28-20231123131906-eqz545r.jpg)

![29.jpg](assets/29-20231123131906-gj8hgpb.jpg)

# Deadlock Handling

A *deadlock* is a cycle of transactions waiting for locks to be released by each other. There are two approaches to handling deadlocks in 2PL: **detection and prevention**.

![30.jpg](assets/30-20231123131906-bambwoh.jpg)

![31.jpg](assets/31-20231123131906-t5yup46.jpg)

## Approach #1:# Deadlock Detection

To detect deadlocks, the DBMS creates a *waits-for* graph where transactions are nodes, and there exists a directed edge from $T_i$ to $T_j$ if transaction $T_i$ is waiting for transaction $T_j$ to release a lock. The system will periodically check for cycles in the waits-for graph (usually with a background thread) and then make a decision on how to break it. **Latches are not needed** when constructing the graph since if the DBMS misses a deadlock in one pass, it will ﬁnd it in the subsequent passes. Note that there is a tradeoff between the frequency of deadlock checks (uses cpu cycles) and the wait time till a deadlock is broken.

When the DBMS detects a deadlock, it will **select a “victim” transaction to abort to break the cycle**. The victim transaction will either restart or abort depending on how the application invoked it.

![32.jpg](assets/32-20231123131906-5d86xe0.jpg)

![33.jpg](assets/33-20231123131906-noin41c.jpg)

![34.jpg](assets/34-20231123131906-acy9p1w.jpg)

![35.jpg](assets/35-20231123131906-77i8jmf.jpg)

![36.jpg](assets/36-20231123131906-iqh8tib.jpg)

The DBMS can consider multiple transaction properties when selecting a victim to break the deadlock:

1. By age (newest or oldest timestamp).
2. By progress (least/most queries executed).
3. By the ## of items already locked.
4. By the ## of transactions needed to rollback with it.
5. # of times a transaction has been restarted in the past (to avoid starvation).

There is no one choice that is better than others. Many systems use a combination of these factors.

After selecting a victim transaction to abort, the DBMS can also decide on how far to rollback the transaction’s changes. It can either rollback the entire transaction or just enough queries to break the deadlock.

![37.jpg](assets/37-20231123131906-kbo23zm.jpg)

![38.jpg](assets/38-20231123131906-na06jcp.jpg)

## Approach #2:# Deadlock Prevention

Instead of letting transactions try to acquire any lock they need and then deal with deadlocks afterwards, deadlock prevention 2PL stops transactions from causing deadlocks before they occur. When a transaction tries to acquire a lock held by another transaction (which could cause a deadlock), the DBMS kills one of them. To implement this, transactions are assigned priorities based on timestamps (older transactions have higher priority). These schemes guarantee no deadlocks because **only one type of direction is allowed** when waiting for a lock. When a transaction restarts, the DBMS **reuses** the same timestamp.

![39.jpg](assets/39-20231123131906-6q4c9gt.jpg)

There are two ways to kill transactions under deadlock prevention:

- **Wait-Die (“Old Waits for Young”)** : If the requesting transaction has a higher priority than the holding transaction, it waits. Otherwise, it aborts.
- **Wound-Wait (“Young Waits for Old”)** : If the requesting transaction has a higher priority than the holding transaction, the holding transaction aborts and releases the lock. Otherwise, the requesting transaction waits.

![40.jpg](assets/40-20231123131906-zqsm9c3.jpg)

![41.jpg](assets/41-20231123131906-b1uhzjy.jpg)

![42.jpg](assets/42-20231123131906-vb56az5.jpg)

# Lock Granularities

If a transaction wants to update one billion tuples, it has to ask the DBMS’s lock manager for a billion locks. This will be slow because the transaction has to take latches in the lock manager’s internal lock table data structure as it acquires/releases locks.

![43.jpg](assets/43-20231123131906-3mj12iw.jpg)

To avoid this overhead, the DBMS can use to use a lock hierarchy that allows a transaction to take more coarse-grained locks in the system. For example, it could acquire a single lock on the table with one billion tuples instead of one billion separate locks. When a transaction acquires a lock for an object in this hierarchy, it implicitly acquires the locks for all its children objects.

![44.jpg](assets/44-20231123131906-jiq1u6j.jpg)

## Database Lock Hierarchy:

1. Database level (Slightly Rare)
2. Table level (Very Common)
3. Page level (Common)
4. Tuple level (Very Common)
5. Attribute level (Rare)

![45.jpg](assets/45-20231123131906-wsk13xc.jpg)

![46.jpg](assets/46-20231123131906-pehfj3e.jpg)

![47.jpg](assets/47-20231123131906-mj8igot.jpg)

**Intention locks** allow a higher level node to be locked in **shared** mode or **exclusive** mode without having to check all descendant nodes. If a node is in an intention mode, then explicit locking is being done at a lower level in the tree.

![48.jpg](assets/48-20231123131906-p0nwrk2.jpg)

- **Intention-Shared (IS)** : Indicates explicit locking at a lower level with shared locks.
- **Intention-Exclusive (IX)** : Indicates explicit locking at a lower level with exclusive or shared locks.
- **Shared+Intention-Exclusive (SIX)** : The sub-tree rooted at that node is locked explicitly in **shared** mode and explicit locking is being done at a lower level with exclusive-mode locks.

![49.jpg](assets/49-20231123131906-6en733w.jpg)

![50.jpg](assets/50-20231123131906-716w06m.jpg)

![51.jpg](assets/51-20231123131906-g97vjz6.jpg)

![52.jpg](assets/52-20231123131906-iuabvik.jpg)

![53.jpg](assets/53-20231123131906-794p0ef.jpg)

![54.jpg](assets/54-20231123131906-7p2u2oz.jpg)

![55.jpg](assets/55-20231123131906-i4qdo18.jpg)

![56.jpg](assets/56-20231123131906-xmr30gj.jpg)

![57.jpg](assets/57-20231123131906-ia3b8mq.jpg)

![58.jpg](assets/58-20231123131906-eml4m4f.jpg)

![59.jpg](assets/59-20231123131906-lu42kd0.jpg)

![60.jpg](assets/60-20231123131906-a808nxd.jpg)

![61.jpg](assets/61-20231123131906-9i37kvr.jpg)

![62.jpg](assets/62-20231123131906-fy8zwc0.jpg)

![63.jpg](assets/63-20231123131906-z8xfwyt.jpg)

![64.jpg](assets/64-20231123131906-8kr0zyh.jpg)

![65.jpg](assets/65-20231123131906-y4amlqc.jpg)

![66.jpg](assets/66-20231123131906-t8473g9.jpg)

![67.jpg](assets/67-20231123131906-a9u96go.jpg)

![68.jpg](assets/68-20231123131906-ut8zy3t.jpg)

![69.jpg](assets/69-20231123131906-s5ynsqi.jpg)

![70.jpg](assets/70-20231123131906-duc53io.jpg)

![71.jpg](assets/71-20231123131906-8f1q3pe.jpg)

![72.jpg](assets/72-20231123131906-ic6ayd6.jpg)
