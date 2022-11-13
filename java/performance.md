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