## Java performance

### 1. Tools 

- **Profilers**

1. Sampling
2. Instrumentation 
3. Native - aynch-profiler, Oracle developer studio

- **JStack**

1. `jstack <process_id>`

- **jcmd**

1. `jcmd <process_id> VM.uptime`
2. `jcmd process_id VM.system_properties`
3. `jcmd process_id VM.version`
4. `jcmd process_id VM.command_line`
5. `jcmd process_id VM.flags [-all]`
6. `jcmd process_id VM.native_memory summary`

- **network usage**
- **disc usage**
- **CPU**

### 2. Java memory 

JVM memory = Heap + Metaspace + CodeCache + (ThreadSize x numberOfThreads) + DirectByteBuffers + JVM native

- metaspace - info about classes and methods, by default no limit for metaspace
- code cache - for JIT
- stacksize - by default 1MB

### 3. Flags

- -Xss (-ThreadStackSize) - stack size
- -Xms - min heap
- -Xmx - max heap
- -XX:MetaspaceSize, -XX:MaxMetaspaceSize
- -XX:HeapDumpOnOutOfMemoryError, -XX:HeapDumpPath
- -XX:+UseStringDeduplication (G1)

### 4. JIT optimizations 

- inlining - method, call graph inlining, tail recursion, 
- removal redundant loads 
- elimination of dead code
- code reordering, splitting, removal
- loop reduction, unrolling or peeling
- exception-directed optimizations 
- switch analysis 
- escape analysis
- synchronization optmization
- Global data flow analyses and optimizations
- GC and memory allocation optimizations

### 5. GCs

- serial  
1. -XX:+UseSerialGC, 
2. single thread, no communication overhead, siuted for single-processor machines
- parallel 
1. -XX:+UseParallelGC, -XX:ParallelGCThreads, -XX:MaxGCPauseMillis, -XX:GCTimeRatio
2. (throughput) throughput over latency 
- G1
1. -XX:+UseG1GC
2. mostly concurrent collector, performs some action concurently with the app, memory divided into 2048 regions
3. -XX:+UseStringDeduplication, -XX:+PrintStringDeduplicationStatistics, -XX:StringDeduplicationAgeThreshold- finds/prints duplications
- ZGC
1. low-latency, awesome to huge heaps, works concurrently, low pauses
2. requires more CPU
2. -XX:+UseZGC, XX:ConcGCThreads
- Shenandoah
1. also a very short pause time, GC concurrent to app, well siuted to applications that needs

### 6. What is NUMA-aware in G1

- Non-Uniform Memory Access
- memory access depends on the memory location relative to processor and for processor local memory access is faster and to non-local memory.
- local memory access takes 30-60 less than non-local memory
- ZGC supports, JDK-6 added it to Parallel, 
