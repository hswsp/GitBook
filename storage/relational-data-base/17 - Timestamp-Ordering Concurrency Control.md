# 17 - Timestamp-Ordering Concurrency Control

![1.jpg](assets/1-20231123131906-kfju16r.jpg)

![2.jpg](assets/2-20231123131906-wczgwl3.jpg)

# Timestamp Ordering Concurrency Control

Timestamp ordering (T/O) is an **optimistic** class of concurrency control protocols where the DBMS assumes that transaction conflicts are rare. Instead of requiring transactions to acquire locks before they are allowed to read/write to a database object, the DBMS instead uses timestamps to determine the serializability order of transactions.

Each transaction $T_i$ is assigned a unique fixed timestamp $TS(T_i )$ that is monotonically increasing. Different schemes assign timestamps at different times during the transaction. Some advanced schemes even assign multiple timestamps per transaction.

If $TS(T_i ) &lt; TS(T_j )$, then the DBMS must ensure that the execution schedule is equivalent to the serial schedule where $T_i$ appears before $T_j$.

![3.jpg](assets/3-20231123131906-njgbh1m.jpg)

There are multiple timestamp allocation implementation strategies. The DBMS can use the system clock as a timestamp, but issues arise with edge cases like daylight savings. Another option is to use a logical counter. However, this has issues with overflow and with maintaining the counter across a distributed system with multiple machines. There are also hybrid approaches that use a combination of both methods.

![4.jpg](assets/4-20231123131906-pngm1gz.jpg)

![5.jpg](assets/5-20231123131906-m3lbknf.jpg)

# Basic Timestamp Ordering (BASIC T/O)

The basic timestamp ordering protocol (BASIC T/O) allows reads and writes on database objects without using locks. Instead, every database object X is tagged with timestamp of the last transaction that successfully performed a read (denoted as $R-TS(X)$) or write (denoted as $W-TS(X)$) on that object. The DBMS then checks these timestamps for every operation. If a transaction tries to access an object in a way which violates the timestamp ordering, the transaction is aborted and restarted. The underlying assumption is that violations will be rare and thus these restarts will also be rare.

![6.jpg](assets/6-20231123131906-99h0mk8.jpg)

## Read Operations

For read operations, if $TS(T_i ) &lt; W-TS(X)$, this violates timestamp order of $T_i$ with regard to the previous writer of $X$ (do not want to read something that is written in the “future”). Thus, $T_i$ is aborted and restarted with a new timestamp. Otherwise, the read is valid and $T_i$ is allowed to read $X$. The DBMS then updates $R-TS(X)$ to be the max of $R-TS(X)$ and $TS(T_i )$. It also has to make a local copy of $X$ in a private workspace to ensure repeatable reads for $T_i$ .

![7.jpg](assets/7-20231123131906-zoz1kdj.jpg)

## Write Operations

For write operations, if $TS(T_i ) &lt; R-TS(X)$ or $TS(T_i ) &lt; W-TS(X)$, $T_i$ must be restarted (do not want to overwrite “future” change). Otherwise, the DBMS allows $T_i$ to write $X$ and updates $W-TS(X)$. Again, it needs to make a local copy of $X$ to ensure repeatable reads for $T_i$ .

![8.jpg](assets/8-20231123131906-kcdvyjf.jpg)

![9.jpg](assets/9-20231123131906-5rm6pzm.jpg)

![10.jpg](assets/10-20231123131906-uhtoarj.jpg)

![11.jpg](assets/11-20231123131906-w433z6j.jpg)

![12.jpg](assets/12-20231123131906-99sp797.jpg)

![13.jpg](assets/13-20231123131906-c9jpu38.jpg)

![14.jpg](assets/14-20231123131906-3j6kus5.jpg)

![15.jpg](assets/15-20231123131906-tytsitq.jpg)

![16.jpg](assets/16-20231123131906-r56x6z2.jpg)

![17.jpg](assets/17-20231123131906-s8a9a2f.jpg)

![18.jpg](assets/18-20231123131906-x6846ob.jpg)

![19.jpg](assets/19-20231123131906-g5bgute.jpg)

![20.jpg](assets/20-20231123131906-ataqisy.jpg)

![21.jpg](assets/21-20231123131906-3t7ryhq.jpg)

![22.jpg](assets/22-20231123131906-o0305on.jpg)

## Optimization: Thomas Write Rule

An optimization for writes is if $TS(T_i ) &lt; W-TS(X)$, the DBMS can instead ignore the write and allow the transaction to continue instead of aborting and restarting it. This is called the **Thomas Write Rule.**  Note that this violates timestamp order of $T_i$ but this is okay because no other transaction will ever read $T_i$ ’s write to object $X$.

The Basic T/O protocol generates a schedule that is conflict serializable if it does not use Thomas Write Rule. It cannot have deadlocks because no transaction ever waits. However, there is a possibility of starvation for long transactions if short transactions keep causing conflicts.

It also permits schedules that are not recoverable. A schedule is *recoverable* if transactions commit only after all transactions whose changes they read, commit. Otherwise, the DBMS cannot guarantee that transactions read data that will be restored after recovering from a crash.

![23.jpg](assets/23-20231123131906-sai1bs7.jpg)

![24.jpg](assets/24-20231123131906-tnoqzf2.jpg)

![25.jpg](assets/25-20231123131906-plmf1sc.jpg)

![26.jpg](assets/26-20231123131906-s0qhky1.jpg)

![27.jpg](assets/27-20231123131906-rcza924.jpg)

![28.jpg](assets/28-20231123131906-qr6kome.jpg)

## **Potential Issues:**

• High overhead from copying data to transaction’s workspace and from updating timestamps.

• Long running transactions can get starved. The likelihood that a transaction will read something from a newer transaction increases.

• Suffers from the timestamp allocation bottleneck on highly concurrent systems.

![29.jpg](assets/29-20231123131906-n1nn78r.jpg)

![30.jpg](assets/30-20231123131906-af3ji72.jpg)

# Optimistic Concurrency Control (OCC)

Optimistic concurrency control (OCC) is another optimistic concurrency control protocol which also uses timestamps to validate transactions. OCC works best when the number of conflicts is low. This is when either all of the transactions are read-only or when transactions access disjoint subsets of data. If the database is large and the workload is not skewed, then there is a low probability of conflict, making OCC a good choice.

In OCC, the DBMS creates a *private workspace* for each transaction. All modifications of the transaction are applied to this workspace. Any object read is copied into workspace and any object written is copied to the workspace and modified there. No other transaction can read the changes made by another transaction in its private workspace.

When a transaction commits, the DBMS compares the transaction’s workspace write set to see whether it conflicts with other transactions. If there are no conflicts, the write set is installed into the “global” database.

![31.jpg](assets/31-20231123131906-4raqegu.jpg)

OCC consists of three phases:

1. **Read Phase**: Here, the DBMS tracks the read/write sets of transactions and stores their writes in a private workspace.
2. **Validation Phase:**  When a transaction commits, the DBMS checks whether it conflicts with other transactions.
3. **Write Phase:**  If validation succeeds, the DBMS applies the private workspace changes to the database.

**Otherwise, it aborts and restarts the transaction.**

![32.jpg](assets/32-20231123131906-m3mutlu.jpg)

![33.jpg](assets/33-20231123131906-23nj688.jpg)

![34.jpg](assets/34-20231123131906-slfcnyn.jpg)

![35.jpg](assets/35-20231123131906-bpe15uc.jpg)

![36.jpg](assets/36-20231123131906-bgo9806.jpg)

![37.jpg](assets/37-20231123131906-eu23jip.jpg)

![38.jpg](assets/38-20231123131906-pdovnxx.jpg)

![39.jpg](assets/39-20231123131906-lejij2g.jpg)

![40.jpg](assets/40-20231123131906-k649ukd.jpg)

![41.jpg](assets/41-20231123131906-nj5kiqw.jpg)

![42.jpg](assets/42-20231123131906-cpyenlq.jpg)

![43.jpg](assets/43-20231123131906-6tc01vs.jpg)

![44.jpg](assets/44-20231123131906-l8zj3vb.jpg)

## Validation Phase

The DBMS assigns transactions timestamps when they enter the validation phase. To ensure only serializable schedules are permitted, the DBMS checks $T_i$ against other transactions for RW and WW conflicts and makes sure that all conflicts go one way.

• **Approach 1**: Backward validation (from younger transactions to older transactions)

• **Approach 2**: Forward validation (from older transactions to younger transactions)

![45.jpg](assets/45-20231123131906-yfnkt72.jpg)

![46.jpg](assets/46-20231123131906-9ptnn3z.jpg)

![47.jpg](assets/47-20231123131906-tiyc04r.jpg)

![48.jpg](assets/48-20231123131906-r7f1o8u.jpg)

![49.jpg](assets/49-20231123131906-3jaqmg2.jpg)

![50.jpg](assets/50-20231123131906-txsm040.jpg)

Here we describes how forward validation works. The DBMS checks the timestamp ordering of the committing transaction with all other running transactions. Transactions that have not yet entered the validation phase are assigned a timestamp of ∞.

If $TS(T_i ) &lt; TS(T_j )$, then one of the following three conditions must hold:

1. $T_i$ completes all three phases before $T_j$ begins its execution (serial ordering).
2. $T_i$ completes before $T_j$ starts its Write phase, and $T_i$ does not write to any object read by $T_j$ . $WriteSet(T_i ) ∩ ReadSet(T_j ) = ∅$.
3. $T_i$ completes its Read phase before $T_j$ completes its Read phase, and $T_i$ does not write to any object that is either read or written by $T_j$ . $WriteSet(T_i ) ∩ ReadSet(T_j ) = ∅$, and $WriteSet(T_i ) ∩ WriteSet(T_j ) = ∅$.

![51.jpg](assets/51-20231123131906-6qqcvrk.jpg)

![52.jpg](assets/52-20231123131906-054g7ze.jpg)

![53.jpg](assets/53-20231123131906-6mbgype.jpg)

![54.jpg](assets/54-20231123131906-viq6xuh.jpg)

![55.jpg](assets/55-20231123131906-hc0dkv9.jpg)

![56.jpg](assets/56-20231123131906-4ftkxz2.jpg)

![57.jpg](assets/57-20231123131906-m2lhyya.jpg)

![58.jpg](assets/58-20231123131906-temuuks.jpg)

![59.jpg](assets/59-20231123131906-22rh2py.jpg)

![60.jpg](assets/60-20231123131906-qlghx1g.jpg)

![61.jpg](assets/61-20231123131906-5v1asda.jpg)

![62.jpg](assets/62-20231123131906-k4oyono.jpg)

![63.jpg](assets/63-20231123131906-evdb44l.jpg)

![64.jpg](assets/64-20231123131906-ehj663y.jpg)

## Potential Issues:

• High overhead for copying data locally into the transaction’s private workspace.

• Validation/Write phase bottlenecks.

• Aborts are potentially more wasteful than in other protocols because they only occur after a transaction has already executed.

• Suffers from timestamp allocation bottleneck.

![65.jpg](assets/65-20231123131906-6jj9grm.jpg)

# Dynamic Databases

![66.jpg](assets/66-20231123131906-dl372r4.jpg)

![67.jpg](assets/67-20231123131906-hy9iec6.jpg)

![68.jpg](assets/68-20231123131906-i6394c1.jpg)

![69.jpg](assets/69-20231123131906-ipe42sy.jpg)

![70.jpg](assets/70-20231123131906-wpopgc3.jpg)

![71.jpg](assets/71-20231123131906-r61b9pj.jpg)

![72.jpg](assets/72-20231123131906-gnemunl.jpg)

![73.jpg](assets/73-20231123131906-6vwxkf8.jpg)

![74.jpg](assets/74-20231123131906-3pfxaor.jpg)

![75.jpg](assets/75-20231123131906-mymcbkw.jpg)

![76.jpg](assets/76-20231123131906-uvo5lm6.jpg)

![77.jpg](assets/77-20231123131906-slamjpw.jpg)

![78.jpg](assets/78-20231123131906-azaxq5i.jpg)

![79.jpg](assets/79-20231123131906-y7ykwx8.jpg)

![80.jpg](assets/80-20231123131906-bqqsok9.jpg)

![81.jpg](assets/81-20231123131906-l51fa80.jpg)

# Isolation Levels

Serializability is useful because it allows programmers to ignore concurrency issues but enforcing it may allow too little parallelism and limit performance. We may want to use a weaker level of consistency to improve scalability.

Isolation levels control the extent that a transaction is exposed to the actions of other concurrent transactions.

![82.jpg](assets/82-20231123131906-znpr1lg.jpg)

**Anomalies:** 
• **Dirty Read**: Reading uncommitted data.
• **Unrepeatable Reads:**  Redoing a read results in a different result.
• **Phantom Reads**: Insertion or deletions result in different results for the same range scan queries.

**Isolation Levels (Strongest to Weakest):**

1. SERIALIZABLE: No Phantoms, all reads repeatable, and no dirty reads.

2. REPEATABLE READS: Phantoms may happen.

3. READ-COMMITTED: Phantoms and unrepeatable reads may happen.

4. READ-UNCOMMITTED: All anomalies may happen.

![83.jpg](assets/83-20231123131906-ous6spo.jpg)

![84.jpg](assets/84-20231123131906-qf1kjy3.jpg)

![85.jpg](assets/85-20231123131906-xvdar9u.jpg)

![86.jpg](assets/86-20231123131906-8uh25dl.jpg)

The isolation levels defined as part of SQL-92 standard only focused on anomalies that can occur in a 2PL-based DBMS. There are two additional isolation levels:

1. CURSOR STABILITY

   • Between repeatable reads and read committed

   • Prevents **Lost Update** Anomaly.

   • Default isolation level in **IBM DB2**.
2. **SNAPSHOT ISOLATION**

   • Guarantees that all reads made in a transaction see a consistent snapshot of the database that existed at the time the transaction started.

   • A transaction will commit only if its writes do not conflict with any concurrent updates made since that snapshot.

   • Susceptible to **write skew** anomaly.

![87.jpg](assets/87-20231123131906-54xvwiz.jpg)

![88.jpg](assets/88-20231123131906-t32m32k.jpg)

![89.jpg](assets/89-20231123131906-jkvetxq.jpg)

![90.jpg](assets/90-20231123131906-t8dkn6x.jpg)

![91.jpg](assets/91-20231123131906-6hw2cb8.jpg)

![92.jpg](assets/92-20231123131906-i32t35t.jpg)

![93.jpg](assets/93-20231123131906-392gvzv.jpg)
