### 1. OIDs - object identifiers

- 4-byte integers 
- relation between db objects and the respective OIDs are stored in system catalogs
- pd_database (databases), pg_class (heap tables)

### 2. Layout of cluster

- PG_VERSION - file containing a major version
- pg_hba.conf - client authentication
- pg_ident.conf - user name mapping
- postgresql.conf - configuration parameters
- postgresql.auto.conf - file used for storing configurations paramters are set in ALTER SYSTEM
- postmaster.opts - A file recording the command line options the server was last started with
- under base there are directories with names reflecting OIDs
- each table and index whose size is < 1GB is a single file stored under the database directory is belongs to. --with-segsize can change default 1GB
- OIDs not always match relfilenode (base/15351/12345)
- '_fsm' and '_vm' file suffixes are respectively responsible free space and visibility map (indexes only have free space and no visibility map)

### 3. Tablespace

- Storing tables and indexes on different disks or file systems to distribute the load and improve performance
- Storing large tables or indexes on disks with a different performance profile, such as faster disks for frequently accessed data
- Storing tables and indexes belonging to different users in separate tablespaces to manage disk space usage
- default tablespace can be changed in postgresql.conf file

### 4. Pages (blocks)

- default 8192 bytes (8KB)
- numbered from 0 = block numbers
- An empty space between the end of line pointers and the beginning of the newest tuple is referred to as free space or hole
- Page layout
  - header data - 24 bytes
    - pd_lsn - 8 bytes, stores LSN of XLOG written by last change of this page, related to WAL
    - pd_checksum - (9.3 > version)
    - pd_lower, pd_upper - end of line pointers, beginning of the newest heap tuple
    - pd_special - for indexed, in page within tables it points to the end of the page. In the page within indexes, it points to the beginning of special space which is the data area held only by indexes and contains the particular data according to the kind of index types such as B-tree, GiST, GiN, etc.
  - heap tuple - record data
  - line pointer - 4 bytes, holds a pointer to heap tuple, indexing tuples, offset number
- TID - tuple identifier, block number + offset number - used by index, each row has it

### 5. Writing and reading tuples

- writing - pd_lower is increased and pd_upper
- B-tree index scan 
  - index file contains index tuples which composed of index key and TID and TID composes block number and offet


### 6. Indexes

- indexes are special db objects
- each operation on indexed data, whether it be insertion, deletion, or update of table rows, indexes for that table need to be updated too, and in the same transaction
- HOT (Heap-Only Tuples) - update of table fields for which indexes haven't been built does not result in index update

**Indexing engine**

- index scan - fine with only few values
```
postgres=# explain (costs off) select * from t where a = 1;
          QUERY PLAN          
-------------------------------
Index Scan using t_a_idx on t
Index Cond: (a = 1)
(2 rows)
```

- bitmap scan

```
postgres=# explain (costs off) select * from t where a <= 100;
             QUERY PLAN            
------------------------------------
 Bitmap Heap Scan on t
   Recheck Cond: (a <= 100)
   ->  Bitmap Index Scan on t_a_idx
         Index Cond: (a <= 100)
(4 rows)
```

- with index 

```
postgres=# create index on t(b);

postgres=# analyze t;

postgres=# explain (costs off) select * from t where a <= 100 and b = 'a';

                    QUERY PLAN                    
--------------------------------------------------
 Bitmap Heap Scan on t
   Recheck Cond: ((a <= 100) AND (b = 'a'::text))
   ->  BitmapAnd
         ->  Bitmap Index Scan on t_a_idx
               Index Cond: (a <= 100)
         ->  Bitmap Index Scan on t_b_idx
               Index Cond: (b = 'a'::text)
(7 rows)
```

- with sequential scan
```
postgres=# explain (costs off) select * from t where a <= 40000;

       QUERY PLAN      
------------------------
 Seq Scan on t
   Filter: (a <= 40000)
(2 rows)
```

- covering indexes
- INCLUDE - indexes -  allows to include non-key columns which don't affect uniqueness and cannot be used in search predicates, but still can serve index-only scans
  - reduce the size of index
  - Including additional columns in the index can allow the query planner to use the index for more queries, which can improve query performance
  - avoiding too much updates on the 
- NULL
- partial indexes 
- concurrently 

#### Hash index

- hash function gives a number which can use used as index of a reqular array where are located references to TIDs (buckets)
- collision where different datasources produces same hash and  therefore TIDs should be recheckde
- 2 pow 32 = 4 billion values
- If we delete some of indexed rows, pages once allocated would not be returned to the operating system, but will only be reused for new data after VACUUMING. The only option to decrease the index size is to rebuild it from scratch using REINDEX or VACUUM FULL command

#### BTree

- suitable for data that can be sorted - greater , greater or equal, equal
- index, as always, is packed into pages, in leaf pages
- B-trees are balanced 
- B-trees are multi-branched, not only 2, each page (8kB) contains a lot of TIDs (hundreds) and the depth of this is pretty small, 4-5 for large tables
- data is sorted in nondecreasing order

#### GiST
#### SP-GiST
#### GIN
#### BRIN

### 3. Proces and memory

- postgres server process (earlier called as postmaster)- a parent of all processes; allocates shared memory; starts background processes and replication associated process; listens 5432 
- backend process - handles all queries and statements from connected clients; communicated with the clined using TCP connection; operatas only on one database, allows multiple clients connected simultaneously (max_connections - default = 100)
- Background Processes
  - background writer - dirty pages on the shared buffer pool are written to a persistent storage
  - checkpointer
  - autovacuum launcher - t requests to create the autovacuum workers to the postgres server
  - WAL writer - writes and flushes periodically the WAL data on the WAL buffer to persistent storage
  - statistic collector - statistics information such as for pg_stat_activity and for pg_stat_database, etc. is collected
  - logging collector - logs into files
  - archiver - archiving logging is executed
- replication process for streaming replication
- background worker process - implemented by users 

```
pstree -p 9687
```

**Memory**

- local memory area - for each backend process
  - work_mem - Executor uses this area for sorting tuples by ORDER BY and DISTINCT operations, and for joining tables by merge-join and hash-join operations.
  - maintenance_work_mem - maintenance operations like VACUUM, REINDEX
  - temp_buffers - for storing temporary tables
- shared memory are - used by all processes
  - shared buffer pool	- for loading pages of tables and indexes from a persistent storage to this pool
  - WAL buffer - WAL data - XLOG records - transaction log; WAL buffer is a buffering area of the WAL data before writing to a persistent storage
  - commit log	- Commit Log - CLOG - keeps the states of all transactions (in_progress,committed,aborted) used by Concurrency Control (CC) mechanism
  - Sub-areas for the access control mechanisms. (e.g., semaphores, lightweight locks, shared and exclusive locks, etc)
  - Sub-areas for the background processes
  - Sub-areas for transaction processing such as save-point and two-phase-commit.

### 4. Query processing

- parser - text -> parse tree, not checking for example table existence, while is checks on semantics
- analyzer/analyzer - semantic analysis and parse tree -> query tree; checks semantics; relies on metadata in parsenodes.h
- rewriter - query tree -> rules -> query tree; rules comes from pg_rules
- planner - generates plan tree which can in the most effective way executor query tree; plan is created on pure const-based optimization, bit not hints and rule optinizations
- executor - executes query plan accessing tables and indexes

### 5. Const estimation

- consts are estimated by the function in costsize.c and all operations executed by the executor have corresponding const functions, for example cost_seqscan() and cost_index(
- cost start-up - cost expected before the firs tuple is fetched - index scan
- cost run - cost of fetch all tuples
- cost total - sum of above two
- explain command shows for example "Seq Scan on tbl  (cost=0.00..145.00 rows=10000 width=8)" where 0 - start-up and 145 is total
- sequential scan (f=cost_seqscan()) 
  - here start-up = 0
  - run cost = cpu run cost + disc run cost = (cpu_tuple_cost + cpu_operator_cost) x Ntuple + seq_page_cost x Npage
  - here seq_page_cost (1.0), cpu_operator_cost (0.0025) and cpu_tuple_cost (0.01) are set in postgresql.conf
  - Ntuple and Npage are number of typles and pages respectively
  - for example with 10000 typles and 45 pages the 'run cost' = (0.01+0.0025) x 10000 + 1.0x45 = 170.0, and total = 0 + 170.0 = 170.0
- index scan (f=cost_index()) - Nindex,tuple =10000 and Nindex,page =30
  - start-up - is a cost of reading index pages to access the first tuple
  - start-up cost = {ceil(log2(Nindex,tuple) + (Hindex + 1) x 50} x cpu_operator_cost ,where Hindex is a high of index tree
  - start-up cost = {ceil(log2(10000) + (1+1) x 50} x 0.0025 - 0.285
  - run-cost - is a sum of cpu and IO cost of both table and index
  - run-cost = (index cpu cost + table cpu cost) + (index IO cost + table IO cost)
    - index cpu cost = selectivity x Nindex,tuple x (cpu_index_tuple_cost +qual_op_cost), where cpu_index_tuple_cost = 0.005 and qual_op_cost=0.0025, selectivity = is a proportion of the search range of the index by specified WHERE clause
    - table cpu cost = selectivity Ntuple x cpu_tuple_cost
    - index IO cpu = ceil(selectivity x Nindex,tuple) + random_page_cost, where random_page_cost is = 4.0 
- random_page_cost is equal to 4 for HDD it makes sens, but the cost on SSD is smaller, so if postgres works on SSD we should use 1.0 instead of 4.0
- sort  (f=cost_sort())
  - if all tuples to be sorted can be sorted in work_mem and quicksort is used
  - otherwise temporary file is created and merge sort is used
  - start-up cost - cost of sorting target tuples - O(Nsort x log2(Nsort)) ,where Nsort is number of tuples to be sorted 
  - run cost is a cost of reading tuples = O(Nsort)
  - start-up cost = C + comparison_cost x Nsort x N logx(Nsort), where C is a total cost of last scan
  - run cost = cpu_opertor_cost x Nsort

### 6. Plan

- Carry out preprocessing
- estimating the cost and getting the cheapest access path
- create a plan from the cheapest path

- access path is definied in the Path structure in  relation.h
- the planner generates a plan tree from the cheapest path

### 7. Executor

- single table queries - the executor takes thew plan nodes in an order from the end of the plan tree to the root and then invokes the functions that perform the processing of the corresponding nodes
- "Sort Method: external sort  Disk: 10000kB" - means that executor used temp file with 10MB size

### 8. Join operators 
 
- nested loop join
  - default
    - does not need start-up operation
    - run-cost is proportional to the product of the size of the outer and inner tables - O(Nouter x Ninner); cost is always estimated
  - materialized 
    - reduces scanning of inner tables comparing to default, and thus reduces the total cost
    - before  running nested loop join the executor writes the inner table tuples  to the work memoryot temp file 
  - indexed
    - index searching is used when index is created on the column in join condition and index scanning instead of sequential one is used 
  - other variations 
    - an existence of an index of the outer table and its attributes are involved in the join condition, it can be used for the index scanning instead of the sequential scan of the outer table
- merge join
  - default
    - can be used only in natural joins (It automatically matches and combines rows from the tables with the same name of the columns;SELECT * FROM students NATURAL JOIN courses) and equi-joins (It is essentially a type of inner join where the join condition is based on equality - SELECT * FROM students JOIN courses ON students.course_id = courses.course_id;)
    - start-up cost - is a sum of sorting cost on both innet and outer sorts
    - in this case all tuples are stored in the memory and the sorting operation will be able to be carried out in the memory itself ( as in temp files)
  - materiazed
    - same as in materialized nested loop join where inner tuples are located in the memory or temp files
  - other based on indexes
- hash join
  - similarly to merge join can be used only with natural and equi-joins
  - behaves differently dependly on the size of tables; If the target table is small enough (more precisely, the size of the inner table is 25% or less of work_mem), it will be a simple two-phase in-memory hash join; otherwise, the hybrid hash join is used with the skew method
  - in-memory hash join
    - processed in work_mem and hash table are is called batch, which has buckets (number is always 2 POWER n)
    - it has 2 phases - build (all tuples of the inner table are inserted into the batch) and probe (each tuple from outer is compared with inner tuples in the batch)
  - Hybrid Hash Join with Skew
    - it used when inner table tuples cannot be stored in the work_mem
    - multiple batches are prepared where one is located in the work_mem and rest of them in temporary files
    - here build and probes are perfored the same number times as number of batchs
- all above can perform - inner join, left, right outer,full outer join 

### 9. FWD - Foreign Data Wrappers

- accessing foreing servers on remote servers 
- possible to execute joins operation from tables from different servers 

### 10. Concurrency control

- support atomicity and isolation (AI from ACID)

**Techniques**

- Multi-version Concurrency Control (MVCC)
  - each write creates a new version of a data item while retaining the old version 
  - reads - the system selects one of the version to ensure isolation of the individual transaction
  - readers do not block writes and reverse
  - Postresql use variation of MVCC = Snapshot Isolation (SI)
- Strict Two-Phase Locking (S2PL)
  - system blocks readers when a writer acquires an exclusive lock for the item 
- Optimistic Concurrency Control (OCC)

**SI**
- Oracle SI - is implemenet by using rollback segments, when writing a new data item, the old version of the item is written to the rollback segment and subsequently the new item is overwritten to the data area
- PostreSQL - a new data is inserted directly into relevant table page and during reading appropriate version is used to an individual transaction by applying visibility check rules
- SI does not allow anomalies - dirty reads, non-repeatable reads, phantom reads.
- full serialization cannot be achived either because it alows anomalies like write skew and readonly transaction skew 
- write skew - multiple transactions attempt to update the same piece of data simultaneously, resulting in conflicts or inconsistencies. This can occur in databases that use a multi-version concurrency control (MVCC) mechanism, where each transaction has its own view of the data and can make changes without being aware of other concurrent transactions.
- readonly transaction skew - multiple read-only transactions attempt to access the same piece of data simultaneously, resulting in a performance degradation or inconsistencies

**SSI - Serializable Snapshot Isolation**

- detect the serialization anomalies and can resolve the conflicts caused by such anomalies

**Transaction id**

- a transaction begins a unique identifier is given by transaction manager (txid, 32-bit unassigned integer)
- special txid 
  - 0 - invalid
  - 1 - bootstrap - for only init of DB
  - 2 - frozen
- txid space is insufficient (more than aprox 4,2 bilions) in practical systems, PostgreSQL treats the txid space as a circle

**Tuple structure**

- Heap tuples in table pages are classified as a usual data tuple and a TOAST tuple
- fields 
  - t_xmin - holds txid of transaction that inserted this tuple
  - t_xmax - holds txid of transaction that deleted or updated this tuple (if nothing like that happened - 0/INVALID)
  - t_cid - holds the commandId = how many SQL commands were invoked before this command within the current transaction starting from 0
  - t_ctid - holds tuple id that points to itself or a new tuple 

- FSM - free space map for all tables or indexs to the pge which can be inserted it
  - information about free space capacity
  - all FSMs are stored with the suffix 'fsm' 

**Commit log (clog)**

- allocated in the shared memory and is used throughout transaction processing
- transaction statuses: IN_PROGRESS, COMMITTED, ABORTED, and SUB_COMMITTED (has been committed to memory by the database management system, but has not yet been written to disk. This means that the changes made by the transaction are visible to other transactions and will be permanent once they are written to disk. However, in the event of a system failure, the changes may not be persisted)
- The clog comprises one or more 8 KB pages in shared memory and logically forms an array.
- The indices of the array correspond to the respective transaction ids, and each item in the array holds the status of the corresponding transaction id
- When the current txid advances and the clog can no longer store it, a new page is appended
- When PostgreSQL shuts down or whenever the checkpoint process runs, the data of the clog are written into files stored under the pg_xact subdirectory. And when PostgreSQL starts up files are loaded back to clog


**Transaction snapshot**

- a dataset that stored information about whether all transactions are active, at a certain point in time for an individual transaction
- '100:104:100,102' - 99 transaction are inactive and 100 and above are active, 104 - first as-yet-unassigned txid, 100,102 - active txids at the moment of the snapshot and that means 101 and 103 are inactive
- obtained snapshot for the visibility check, active transactions in the snapshot must be treated as in progress even if they have actually been committed or aborted.

### 11. Vacuum Processing

- main task is removal of dead tuples and freezing transaction ids
- vacuum
  - concurrent vacuum - removes dead tuples for each page of the table file
    - remove dead tuples and defragment live tuples of each page, remove dead indexes
    - freeze old txids of tuples if necessary, update frozen tixd releated to system catalogs, removal unnecessary parts of the clog 
    - update FSM and VM of porcessed tables
    - statistics update
  - full vacuum - removes dead tuples and de-fragments live tuples
  - autovacuum daemon was costly therefore visibility map was introduced to improve the efficiency and removal dead tuples

### 12. Visibility map

- reduces cost of vacuuming 
- each table has an individual visibility map that holds the visibility of each page in the table file

### 13. Freeze processing 

- two modes lazy (typically) and eager
- eager scans all pages regardless of whether each contains or not dead tuples, updates also catalogs to freeze

### 14. Heap Only Tuple (HOT)

- for effectively usage of indexes and tables during the update of row if it is stored on the same page that stores old row
- reduces a necessity of vacuum processing
- in HOT postgresql does not insert corresponding index tuple 

### 15. Index-only scans 

- to reduce I/O costs it uses directly index without accessing corresponding table pages then all of the target entries of theselect are included in the index table 
- at first glance that accessing the table pages is not required because the index tuples contain the necessary data. However, in fact, PostgreSQL has to check the visibility of the tuples in principle, and the index tuples do not have any information about transactions such as the t_xmin and t_xmax of the heap tuples. It uses visibility map to solve this dilemma

### 16. Buffer manager
