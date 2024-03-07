# 15 - Concurrency Control Theory

![1.jpg](assets/1-20231123131905-n91ttus.jpg)

![2.jpg](assets/2-20231123131905-9d5k9kn.jpg)

# Motivation

- **Lost Update Problem** (**Concurrency Control**): How can we avoid race conditions when updating records at the same time?
- **Durability Problem** (**Recovery**): How can we ensure the correct state in case of a power failure?

![3.jpg](assets/3-20231123131905-wmwl8tn.jpg)

![4.jpg](assets/4-20231123131905-yyu5y45.jpg)

![5.jpg](assets/5-20231123131905-q79qp2o.jpg)

# Transactions

A *transaction* is the execution of a sequence of one or more operations (e.g., SQL queries) on a shared database to perform some higher level function. They are the basic unit of change in a DBMS. Partial transactions are not allowed (i.e. transactions must be atomic).

Example: Move $100 from Andy’s bank account to his promotor’s account

1. Check whether Andy has $100.
2. Deduct $100 from his account.
3. Add $100 to his promotor’s account.

Either all of the steps need to be completed or none of them should be completed.

![6.jpg](assets/6-20231123131905-ji2j8to.jpg)

![7.jpg](assets/7-20231123131905-ekt136j.jpg)

## The Strawman System

A simple system for handling transactions is to execute one transaction at a time **using a single worker** (e.g. one thread). Thus, only one transaction can be running at a time. To execute the transaction, the DBMS copies the entire database ﬁle and makes the transaction changes to this new ﬁle. If the transaction succeeds, then the new ﬁle becomes the current database ﬁle. If the transaction fails, the DBMS discards the new ﬁle and none of the transaction’s changes have been saved. This method is slow as it does not allow for concurrent transactions and requires copying the whole database ﬁle for every transaction.

A (potentially) better approach is to allow concurrent execution of independent transactions while also maintaining correctness and fairness (as in all transactions are treated with equal priority and don’t get ”starved” by never being executed). But executing concurrent transactions in a DBMS is challenging. It is difﬁcult to ensure correctness (for example, if Andy only has $100 and tries to pay off two promoters at once, who should get paid?) while also executing transactions quickly (our strawman example guarantees sequential correctness, but at the cost of parallelism).

![8.jpg](assets/8-20231123131905-tx7w6o0.jpg)

![9.jpg](assets/9-20231123131905-sez0qt8.jpg)

Arbitrary interleaving of operations can lead to:

- **Temporary Inconsistency:**  Unavoidable, but not an issue.
- **Permanent Inconsistency**: Unacceptable, cause problems with correctness and integrity of data.

The scope of a transaction is only inside the database. It cannot make changes to the outside world because it cannot roll those back. For example, if a transaction causes an email to be sent, this cannot be rolled back by the DBMS if the transaction is aborted.

![10.jpg](assets/10-20231123131905-4zbkbep.jpg)

# Deﬁnitions

Formally, a *database* can be represented as a ﬁxed set of named data objects $(A, B, C, . . .)$. These objects can be attributes, tuples, pages, tables, or even databases. The algorithms that we will discuss work on any type of object but all objects must be of the same type.

![11.jpg](assets/11-20231123131905-01pi9yg.jpg)

A transaction is a sequence of read and write operations (i.e., $R(A)$, $W(B)$) on those objects. To simplify discussion, this deﬁnition assumes the database is a ﬁxed size, so the operations can only be reads and updates, not inserts or deletions.

![12.jpg](assets/12-20231123131905-tuhpmi0.jpg)

The boundaries of transactions are deﬁned by the client. In SQL, a transaction starts with the `BEGIN` command. The outcome of a transaction is either `COMMIT` or `ABORT`. For `COMMIT`, either all of the transaction’s modiﬁcations are saved to the database, or the DBMS overrides this and aborts instead.

For `ABORT`, all of the transaction’s changes are undone so that it is like the transaction never happened. Aborts can be either self-inﬂicted or caused by the DBMS.

![13.jpg](assets/13-20231123131905-t4h1gec.jpg)

The criteria used to ensure the correctness of a database is given by the acronym **ACID**.

- **A**tomicity: Atomicity ensures that either all actions in the transaction happen, or none happen.
- **C**onsistency: If each transaction is consistent and the database is consistent at the beginning of the transaction, then the database is guaranteed to be consistent when the transaction completes.
- **I**solation: Isolation means that when a transaction executes, it should have the illusion that it is isolated from other transactions.
- **D**urability: If a transaction commits, then its effects on the database should persist.

![14.jpg](assets/14-20231123131905-yme9pte.jpg)

![15.jpg](assets/15-20231123131905-wzt57iv.jpg)

# ACID: Atomicity

The DBMS guarantees that transactions are **atomic**. The transaction either executes all its actions or none of them. There are two approaches to this:

![16.jpg](assets/16-20231123131905-og0wr2z.jpg)

![17.jpg](assets/17-20231123131905-ep68b5r.jpg)

## Approach #1:# Logging

DBMS logs all actions so that it can undo the actions of aborted transactions. It maintains undo records both in memory and on disk. Logging is used by almost all modern systems for audit and efﬁciency reasons.

![18.jpg](assets/18-20231123131905-9d8ngfl.jpg)

## Approach #2:# Shadow Paging

The DBMS makes copies of pages modiﬁed by the transactions and transactions make changes to those copies. Only when the transaction commits is the page made visible. This approach is typically slower at runtime than a logging-based DBMS. However, one beneﬁt is, if you are only single threaded, there is no need for logging, so there are less writes to disk when transactions modify the database. This also makes recovery simple, as all you need to do is delete all pages from uncommitted transactions. In general, though, better runtime performance is preferred over better recovery performance, so this is rarely used in practice.

![19.jpg](assets/19-20231123131905-kr8ne74.jpg)

# ACID: Consistency

At a high level, consisitency means the “world” represented by the database is **logically** correct. All questions (i.e., queries) that the application asks about the data will return logically correct results. There are two notions of consistency:

**Database Consistency**: The database accurately represents the real world entity it is modeling and follows integrity constraints. (E.g. The age of a person cannot not be negative). Additionally, transactions in the future should see the effects of transactions committed in the past inside of the database.

**Transaction Consistency**: If the database is consistent before the transaction starts, it will also be consistent after. Ensuring transaction consistency is the **application’s** responsibility.

![20.jpg](assets/20-20231123131905-9m8ffvi.jpg)

![21.jpg](assets/21-20231123131905-lzxi5rf.jpg)

![22.jpg](assets/22-20231123131905-799ibt6.jpg)

# ACID: Isolation

The DBMS provides transactions the illusion that they are running alone in the system. They do not see the effects of concurrent transactions. This is equivalent to a system where transactions are executed in serial order (i.e., one at a time). But to achieve better performance, the DBMS has to interleave the operations of concurrent transactions while maintaining the illusion of isolation.

![23.jpg](assets/23-20231123131905-vhabrxw.jpg)

## Concurrency Control

A *concurrency control protocol* is how the DBMS decides the proper interleaving of operations from multiple transactions at runtime.

There are two categories of concurrency control protocols:

1. **Pessimistic**: The DBMS assumes that transactions will conﬂict, so it doesn’t let problems arise in the ﬁrst place.
2. **Optimistic**: The DBMS assumes that conﬂicts between transactions are rare, so it chooses to deal with conﬂicts when they happen after the transactions commit.

![24.jpg](assets/24-20231123131905-u7rrjmg.jpg)

![25.jpg](assets/25-20231123131905-96p284z.jpg)

![26.jpg](assets/26-20231123131905-f0mo8rd.jpg)

![27.jpg](assets/27-20231123131905-pbnosl4.jpg)

![28.jpg](assets/28-20231123131905-iniiru6.jpg)

![29.jpg](assets/29-20231123131905-abodlli.jpg)

![30.jpg](assets/30-20231123131905-rwk98uc.jpg)

![31.jpg](assets/31-20231123131905-kg0s7xq.jpg)

![32.jpg](assets/32-20231123131905-cxky2kr.jpg)

![33.jpg](assets/33-20231123131905-f4x1isj.jpg)

![34.jpg](assets/34-20231123131905-0d4ibcz.jpg)

![35.jpg](assets/35-20231123131905-sw7vo5e.jpg)

The order in which the DBMS executes operations is called an *execution schedule*. We want to interleave transactions to maximize concurrency while ensuring that the output is “correct”. The goal of a concurrency control protocol is to generate an execution schedule that is is **equivalent to some serial execution**:

- **Serial Schedule**: Schedule that does not interleave the actions of different transactions.
- **Equivalent Schedules**: For any database state, if the effect of execution the ﬁrst schedule is identical to the effect of executing the second schedule, the two schedules are equivalent.
- **Serializable Schedule**: A serializable schedule is a schedule that is equivalent to any serial execution of the transactions. Different serial executions can produce different results, but all are considered “correct”.

![36.jpg](assets/36-20231123131905-sh6rzjt.jpg)

![37.jpg](assets/37-20231123131905-5dl4gfk.jpg)

A *conﬂict* between two operations occurs if the operations are for different transactions, they are performed on the same object, and at least one of the operations is a write. There are three variations of conﬂicts:

- **Write-Read Conﬂicts** (“**Dirty Reads**”): A transaction sees the write effects of a different transaction before that transaction committed its changes.
- **Read-Write Conﬂicts** (“**Unrepeatable Reads**”): A transaction is not able to get the same value when reading the same object multiple times.
- **Write-Write conﬂict** (“**Lost Updates**”): One transaction overwrites the uncommitted data of another concurrent transaction.

There are two types for serializability: (1) *conﬂict* and (2) *view*. Neither deﬁnition allows all schedules that one would consider serializable. In practice, DBMSs support conﬂict serializability because it can be enforced efﬁciently.

![38.jpg](assets/38-20231123131905-4dbdgb5.jpg)

![39.jpg](assets/39-20231123131905-7c8ls1c.jpg)

![40.jpg](assets/40-20231123131905-ezt29rk.jpg)

![41.jpg](assets/41-20231123131905-a4yt0l8.jpg)

![42.jpg](assets/42-20231123131905-t4ntn1b.jpg)

## Conﬂict Serializability

Two schedules are ***conﬂict equivalent*** iff they involve the same operations of the same transactions and every pair of conﬂicting operations is ordered in the same way in both schedules. A schedule $S$ is ***conﬂict serializable*** if it is conﬂict equivalent to some **serial schedule**.

One can verify that a schedule is conﬂict serializable by swapping non-conﬂicting operations until a serial schedule is formed. For schedules with many transactions, this becomes too expensive. A better way to verify schedules is to use a *dependency graph* (*precedence graph*).

![43.jpg](assets/43-20231123131905-47ghht5.jpg)

![44.jpg](assets/44-20231123131905-7uy9wd3.jpg)

![45.jpg](assets/45-20231123131905-ffli3h6.jpg)

![46.jpg](assets/46-20231123131905-e2xcr1z.jpg)

![47.jpg](assets/47-20231123131905-xdbe39v.jpg)

![48.jpg](assets/48-20231123131905-4rl4kez.jpg)

![49.jpg](assets/49-20231123131905-0drog4s.jpg)

![50.jpg](assets/50-20231123131905-lt2f6sh.jpg)

In a dependency graph, each transaction is a node in the graph. There exists a directed edge from node $T_i$  to $T_j$ iff an operation $O_i$ from $T_i$ conﬂicts with an operation $O_j$ from $T_j$ and $O_i$ occurs before $O_j$ in the schedule. Then, a schedule is conﬂict serializable iff the dependency graph is acyclic.

![51.jpg](assets/51-20231123131905-jaht5zn.jpg)

![52.jpg](assets/52-20231123131905-8jx0q51.jpg)

![53.jpg](assets/53-20231123131905-595bz0i.jpg)

![54.jpg](assets/54-20231123131905-gux4av5.jpg)

![55.jpg](assets/55-20231123131905-4jpqa2x.jpg)

![56.jpg](assets/56-20231123131905-2kk2cz2.jpg)

![57.jpg](assets/57-20231123131905-49gwk5a.jpg)

![58.jpg](assets/58-20231123131905-5z4mpcz.jpg)

![59.jpg](assets/59-20231123131905-5zx6ije.jpg)

![60.jpg](assets/60-20231123131905-fjkt7gl.jpg)

![61.jpg](assets/61-20231123131905-eyica6x.jpg)

![62.jpg](assets/62-20231123131905-773uvcc.jpg)

![63.jpg](assets/63-20231123131905-tvo8to4.jpg)

## View Serializability

*View serializability* is a weaker notion of serializibility that allows for all schedules that are conﬂict serializable and “**blind writes**” (i.e. performing writes without reading the value ﬁrst). Thus, it allows for more schedules than conﬂict serializability, but is difﬁcult to enforce efﬁciently. This is because the DBMS does not know how the application will “interpret” values.

![64.jpg](assets/64-20231123131905-vm0hikn.jpg)

![65.jpg](assets/65-20231123131905-j3clhdd.jpg)

![66.jpg](assets/66-20231123131905-x57fycs.jpg)

![67.jpg](assets/67-20231123131905-4tvdwkf.jpg)

![68.jpg](assets/68-20231123131905-0wj4tjb.jpg)

## Universe of Schedules

$SerialSchedules ⊂ ConflictSerializableSchedules ⊂ V iewSerializableSchedules ⊂ AllSchedules$

![69.jpg](assets/69-20231123131905-i441mtj.jpg)

# ACID: Durability

All of the changes of committed transactions must be **durable** (i.e., persistent) after a crash or restart. The DBMS can either use logging or shadow paging to ensure that all changes are durable.

![70.jpg](assets/70-20231123131905-7keab0m.jpg)

![71.jpg](assets/71-20231123131905-j1if857.jpg)

![72.jpg](assets/72-20231123131905-txu9ppq.jpg)

![73.jpg](assets/73-20231123131905-b9yz5cz.jpg)

![74.jpg](assets/74-20231123131905-amg2p77.jpg)

![75.jpg](assets/75-20231123131905-uacrtmy.jpg)

![76.jpg](assets/76-20231123131905-to41bj9.jpg)

![77.jpg](assets/77-20231123131905-s02bmb4.jpg)

![78.jpg](assets/78-20231123131905-s0u27lf.jpg)

![79.jpg](assets/79-20231123131905-lippu4t.jpg)

![80.jpg](assets/80-20231123131905-aqdrhpc.jpg)
