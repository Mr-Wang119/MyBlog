---
title: 'Learning Notes: Google File System'
date: 2022-03-12 10:55:54
tags: Distributed System
cover: https://s3.amazonaws.com/images.masen.com/2022/03/0d54a6da76caa99f283ec1b8fc9ef867.jpg
---

> https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/gfs-sosp2003.pdf

Google File System is a scalable distributed file system for large distributed data-intensive applications. 

# Characteristic

----

- Fault tolerance;
- Inexpensive commodity hardware;
- Supporting distributed applications.

<!--more-->

# Key Observations

---

## 1. Component failures are the norm rather than the exception.

Constant monitoring, error detection, fault tolerance, and automatic recovery must be integral to the system. 

## 2. Files are huge by traditional standards. Multi-GB files are common.

Design assumptions and parameters such as I/O operation and block size have to be revisited.

## 3. Most files are mutated by appending new data rather than overwriting existing data

Appending becomes the focus of performance optimization and atomicity guarantees.

## 4. Co-designing the applications and the file system API benefits the overall system by increasing our flexibility.

The related consistency model vastly simplifies the file system without imposing an onerous burden on the applications. Introduced an atomic append operation so that multiple clients can append concurrently to a file without extra synchronization between them.

# Assumptions

---

- System often fails
- System stores a modest number of large files
- Read:
  - Large streaming reads
  - Small random reads
- Many large, sequential writes that append data to files
- Multiple clients concurrently append to the same file
- High sustained bandwidth is more important than low latency

# Architecture

---

![image-20220312112220484](https://s3.amazonaws.com/images.masen.com/2022/03/2133ea129f3ff777f472c343f4c4aa0f.png)

A GFS cluster consists of a single master and multiple chunkservers and is accessed by multiple clients. Files are divided into fixed-size chunks. Each chunk is replicated on multiple chunkservers. The master maintains all file system metadata. Clients interact with the master for metadata operations, but all data-bearing communication goes directly to the chunkservers. Neither the client nor the chunk server caches file data. Client caches offer little benefit because most applications stream through huge files or have working sets too large to be cached.

## Data Flow

![image-20220312170638648](https://s3.amazonaws.com/images.masen.com/2022/03/436fc1dc4d0e409aca9210b87fc6602f.png)

Utilize each machine's network bandwidth: The data is pushed linearly along a chain of chunk servers rather than distributed in some other topology.

Avoid network bottlenecks and high-latency links as much as possible: Each machine forwards the data to the "closest" machine in the network topology that has not received it.

Minimize latency: Pipelining the data transfer over TCP connections as using a switched network with full-duplex links.

The ideal elapsed time for transferring B bytes to R replicas is B/T + RL where T is the network throughput and L is the latency to transfer bytes between two machines.

## Master

Maintain: File and chunk namespaces, access control information, the mapping from files to chunks, and the current locations of chunks.

Control: System-wide activities such as chunk lease management, garbage collection of orphaned chunks, and chunk migration between chunkservers.

Single Server Advantages: Simplify the design and enable the master to make sophisticated chunk placement and replication decisions using global knowledge.

No Single Server bottleneck: (1) Clients never ask for files or write file data from the master. (2) Clients cache this information for a limited time. In this way, they could interact with chunkservers directly.

## Chunk Size

64 MB. Lazy space allocation avoids wasting space due to internal fragmentation.

Large Chunk Size Advantages: (1) Reduce the client's need to interact with the master. (2) Client could perform many operations on a given chunk, reducing network overhead by keeping a persistent TCP. (3) Reduce master meta data size. In this way, meta data could be put into the memory of the master.

Large Chunk Size Disadvantages: Small file size means a small number of chunks, perhaps just one. The chunkservers storing those chunks may become hot spots if many clients are accessing the same file. The solution is: (1) store this with a higher replication factor, (2) stagger app start times.

## Meta Data

- File and chunk namespaces
- Mapping from files to chunks
- Locations of each chunk's replicas.

All metadata is kept in the master's memory. The first two types are also kept persistent by logging mutations to an operation log stored on the master's local disk and replicated on remote machines.

Store in Memory Advantages: It is easy and efficient for the master to periodically scan through its entire state in the background. (Chunk garbage collection, re-replication in the presence of chunkserver failures, and chunk migration to balance load and disk space usage across chunkservers)

What if memory limitation? The master maintains less than 64 bytes of metadata for each 64 MB chunk. The file namespace data typically requires less than 64 bytes per file because it stores file names compactly using prefix compression.

## Chunk Location

Not persistent record. The master polls at startup.

Updated: The master controls all chunk placement and monitors chunkserver status with regular HeartBeat messages.

## Operation Log

Historical records of critical metadata changes. Logical time line that defines the order of concurrent operations.

(1) Replicate it on multiple remote machines.

(2) Respond to a client operation only after flushing the corresponding log record to disk both locally and remotely.

Minimize startup time:  Keep the log small. The master checkpoints its state whenever the log grows beyond a certain size so that it can recover by loading the latest checkpoint from local disk and replying only the limited number of log records after that.

## Consistency Model

File namespace mutations are atomic. Namespace locking guarantees atomicity and correctness.

![image-20220312161707539](https://s3.amazonaws.com/images.masen.com/2022/03/ba7d8985b14b499fd1ce051c4cfeef8a.png)

Defined: Consistent and clients will see what the mutation writes in its entry.

Consistent but undefined: All clients will always see the same data, but it may not reflect what anyone mutation has written.

After a sequence of successful mutations, the mutated file region is guaranteed to be defined and contain the data written by the last mutation.

​	(a) applying mutations to a chunk in the same order on all its replicas.

​	(b) Using chunk version numbers to detect any replica that has become stale because it has missed mutations while its chunkserver was down.

❌ Stale replica:

​	(a) The cache entry has a timeout and the next open of the file, which purges from the cache all chunk information for the file.

​	(b) As most of the files are append-only, a stale replica usually returns a premature end of chunk rather than outdated data.	

❌ Component failures: GFS identifies failed chunkservers by regular handshakes between master and all chunkservers and detects data corruption by checksumming.

## Implications for Applications

- Appends rather than overwrites
- Checkpoints include application-level checksums
- Writing self-validation
- Self-identifying records

# Atomic Record Appends

---

In a record append, the client specifies only the data. GFS appends it to the file at least once atomically.

Similar to the data flow, except if append cause the chunk exceeds maximum size (64 MB).

# Snapshot

---

A copy of a file or a directory tree almost instantaneously. Use standard copy-on-write techniques to implement snapshots. 

(1) The master revokes any outstanding leases on the chunks in the files it is about to snapshot.

(2) The master logs the operation to disk. It then applies this log record to its in-memory state by duplicating the metadata for the source file or directory tree. The newly created snapshot files point to the same chunks as the source files.

(3) The client wants to write to Chunk C after snapshot operation. It sends a request to the master to find the lease handler. The master checks that the reference count for chunk C is greater than one. It would pick a new chunk handle.

# Master Operations

- Allow multiple operations to be active and use locks over regions of the namespace to ensure proper serialization.
- Use lookup table: fullPathNames -> metaData
- Read lock: Prevent the directory from being deleted, renamed, or snapshotted.
- Write lock: Prevent creating a file with the same name.

```
/d1/d2/.../dn/     leaf
  read-locks    write-locks
```

# Chunk Replicas

## Creation

- Place on chunkservers with below-average disk space utilization.
- Limit the number of "recent" creations on each chunkserver.
- Spend replicas of a chunk across racks.

## Re-replicates

The master re-replicates a chunk as soon as the number of available replicas falls below a user-specific goal.

- A chunkserver becomes unavailable.
- Replica corrupted.
- One of the disks is disabled.
- Replication goal is increased.

Priority: (1) How far to the goal (2) is there any chunk that is blocking client progress.

## Stale Replica Detection

Master maintains a chunk version number to distinguish between up-to-date and stale replicas. After granting a new lease, the master increases the chunk version number and informs the up-to-date replicas. After the chunkserver restarts and reports its set of chunks and associated version number. If the master sees a version number greater than the one in its records, the master assumes that it failed when granting the lease.
