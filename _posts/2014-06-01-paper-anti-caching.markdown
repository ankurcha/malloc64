---
layout: post
title: "Paper: Anti-caching: A new approach to database management system architecture"
date: 2014-06-01
categories: paper databases vldb
---

Traditionally, databases have involved heavily encoded disk storage format + buffer pool for caching hot segments of data. Executing query
first checks the buffer pool and if data is not present there an eviction occurs for the disk block needed and query resumes. This involves
substantial overhead in maintaining the buffer pool. In some cases almost 1/3rd of the CPU (if all data exists in the buffer pool).


## **ALT** Main memory databases

Main memory databases store all data in memory and do not have a buffer pool. => drastic improvement in performance but it also needs all
of the data to fit in memory otherwist the OS will pagefault when we try to access data that is not in physical memory (virtual memory paging) i.e.
page faults. This causes all txns to be stalled.

## **ALT** Distributed Cache

This is a widely adopted stratefy wherein we use a main memory distributed cache (eg: memcached) in front of a regular disk backed DBMS. But,

* Objects are double buffered (DB buffer pool + distributed cache)
* Requires apps to embed logic to update/maintain/invalidate cache.

## **Intro** Anti-caching

DBMS runs with the data in memory and when memory is exhausted, it evicts the coldest tuples from memory to disk with minimal encoding. => 'hottest'
data resides in memory and 'colder' data is on disk. *Data is either on disk or in memory but never in both places*.

* Data starts off in memory and cold data is evicted to disk.
* Allows for fine-grained control (at tuple level) for evictions.
* Non-blocking fetches: When a txn needs a block that is not in memory, it is aborted, the tuples are fetched to memory (and evictions may happen to accommodat this) and the txn is restarted when the blocks are available. Meanwhile, all other txns continue.
* We can batch disk block reads so that multiple disk blocks can be read together - increases performance/throughput.

Anticaching architecture out performs traditional disk-based and hybrid architecture for popular OLTP workloads.

## Assumptions

* Restricts the scope of queries to fit in main memory.
* All indexes fit in main memory.

## H-STORE system overview

Traditional DBMS - if buffer pool is full, the dbms chooses a block to evict and make space for the incoming dbms one (from disk) - needs concurrency control mechanisms to allow other txns to continue while the stalled one is waiting.

Recently, RAM is cheap enough to store all or most of the dataset in memory for most OLTP workloads. - This is the scenario H-Store attempts to target

Components of H-Store
1. H-Store node - single computer that manages multiple partitions.
2. Partition - a disjoint subset of the data. Each partition is assigned a single threaded execution engine that executes txns and queries.
3. Hstore can execute adhoc queries but it is primarily targeted to work with stored procedures. -> a txn is an invocation of a stored procedure

Stored procedure has *control code* that invokes predifined parameterized sql code.

### Workload ->

**Single partition transactions**
1. Most txns are local to a single node.
2. Examined in the user-space hstore client, params are substituted to form a runnable query, so the txn can be sent to the correct node where it is executed completely.

**Multi-partitio transactions**
Consists of multiple phases in which more than one partition is touched.

Each transaction is given a unique txn identifier based on the time it arrived into the system. If a txn with a higher transaction id has been already executed, incoming transaction is rejected.

Multipartiton transactions use an extension of this protocol where each local executor cannot run other transactions until the multipartition transaction is completed.

Each DBMS node continuously writes async snapshots of the database to the disk at fixed intervals. Between these intervals, it writes out a record, to a *command log*, of each txn that completes successfully.

## Anticaching system model

The disk is used as a place to spill cold tuples when size of the database exceeds the size of main memory. **A tuple is never copied**. It either lives in memory or on disk based on anti-cache.

The DBMS evicts cold data to the anti-cache to make space for new data -- constructs fixed sized blocks for LRU tuples to be sent to the anti-cache (disk).

When a txn needs an evicted block, it switches to pre-pass mode to learn about all the blocks that the txn needs. The txn is then aborted (rollback changes if needed) and holds it while tuples are fetched into memory in the background. Once the data has been merged back to the memory resident data, txn is restarted. Other txns keep executing while data is being fetched from disk.

### Storage Architecture

Storage manager (in each partition) contains:

* Disk resident hash table that stores evicted blocks of tuples called the **Block Table**.
* In-memory **Evicted table** that maps evicted tuples to block ids.
* In-memory **LRU chain** of tuples for each table.

These structures are all single threaded so no concurrency control mechanisms are needed.

**Currently, we require that all the primary key and the secondary indexes fit in memory**.

**Block Table**: A hash table that maintains the blocks of tuples that have been evicted from the DBMS's main memory storage. Each block is the same fixed-size and is assigned a unique 4-byte key.
A block header contains the identifier for the single table that its tuples were evicted from and timestamp for the block creation time. Body containst eh serialized evicted tuples from a single table. Each evicted tuple is prefixed with its size and is serialized in a format that closely resembles the in-memory format. The key portion of the Block Table stays in memory but the values are stored on disk without OS or filesystem caching.

**Evicted Table**: Keeps track of the tuples that have been evicted out to disk. Each evicted tuple is assigned a 4-byte identifies that corresponds to its offset in the block it resides in. The dbms updates any indexes containing evicted tuples to reference the Evicted Table.

**LRU Chain**: Allows the DBMS to quickly determine at runtime th least recently used tuples to combine into a new block to evict. LRU chain is a doubly linked list where each tuple points to the next and previous most recently used tuple for its table. The dbms embeds pointers directly in the tuples' headers.  The pointer for each tuple is a 4-byte offset of that record in its table's memory at the partition (instead of an 8-byte memory location). The DBMS selects a fraction of the txn to monitor at runtime. Because hot tuples are by definition accessed more frequently and thus more likely to be updated in the LRU chain. The rate at which transactions are sampled can be tuned by a parameter - 0 < \alpha < 1. Some tables can be specifically marked as evictable during schema creation. Any table not marked as evictable will not be maintained in the LRU chain and will remain entirely in memory.

### Block Retrieval

The system first issues a non-blocking read to retrieve the blocks from disk. This allows other transaction to continue while block is being read off the disk. Any transaction that attempts to read an evicted tuple in these blocks is aborted while it is still on disk. Once the requested blocks are retrieved, the aborted transaction(s) is rescheduled. Before it starts, the DBMS performs a "stop-and-copy" operation whereby all transactions are blocked at that partition while the unevicted tuples are merged from the staging buffer into the regular table storage. Then removes all of th entries for these tuples in the evicted table and then updates the table's indexes to point to the real tuples. We may merge either the whole block (Block-Merging) or just the tuples needed (Tuple-Merging).

**Block Merging**: Merge all the tuples from the retrieved block into memory, may add tuples that won't ever be needed by transactions. This can either lead to continuous un-eviction/re-eviction cycles (and be detrimental) or can cause tuples that may eventually be needed anyway ( and avoid more stop-load-merge operations).

**Tuple-Merging**: Only merge the tuples that are necessary for the transaction. Once the desired tuples are merged, the fetched block is discarded. THis can lead to a lot of wasted effort if only a small number of tuples are merged and subsequent transactions cause the same set of blocks to be loaded and reloaded. This can lead to holes in the Evicted blocks which should be *compacted lazily* using some lazy block compaction algorithm during the merge process.

### Distributed Transactions

H-Store will switch a distributed txn into the "pre-pass" mode just as a single partition txn when it attempts to access evicted tuples at any one of its partition. The txn is aborted and not requeued until it receives a notification that all of the blocks that it needs have been retrieved from the nodes in the cluster.

### Snapshots & Recovery

In main-memory DBMS, we use snapshots and command logging [22, 29] are used. The DBMS serializes all the contents of the regular tables and index data, as well as the contents of the Evicted Table, and writes it to disk. At the same time, the DBMS also makes a copy of the Block Table on disk as it existed when the snapshot began. No evictions are allowed to occur while the snapshotting is in progress.
To recover from a crash, the DBMS loads the last snapshot from disk, then replays the txns from the command log that were created after this snapshot was taken.

To keep the size of snapshots small, the DBMS takes **delta snapshots**. These delta-snapshots may be collapsed at regular intervals to avoid keeping a large number of deltas.

## Results / Evaluation
* For highly-skewed workloads ( workloads with skews of 1.5 and 2.5 ) the anti-caching architecture outperforms MySQL by a factor of 9x for readonly, 18x for read-heavy and 10x on write-heavy workloads for datasets 8x memory.
* For hybrid MySQL + memcached architecture, by a factor of 2x for read-only, 4x for read-heavy and 9x for write-heavy workloads for 8x memory.

The lower performance for workloads in hybrid-MySQL architecture is due to the overhead of synchronizing values in Memcached and in MySQL in event of a write. For lower skew, there is a high cost of cache miss in this hybrid architecture.
