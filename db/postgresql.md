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
