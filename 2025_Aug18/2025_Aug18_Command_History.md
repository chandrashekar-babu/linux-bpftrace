#### Useful one-liner scripts using bpftrace

1. **`watch grep ^Vm /proc/172386/status`**  
   *(Not bpftrace) Uses watch to monitor memory usage of process ID 172386*

2. **`bpftrace -l`**  
   *Lists all available probes (tracepoints, kprobes, uprobes, etc.)*

3. **`bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s: Opened %s\n", comm, args->filename); }'`**  
   *Prints process name and filename whenever openat() syscall is entered*

4. **`bpftrace -e 'tracepoint:syscalls:sys_enter_open* { @[comm] = count(); }'`**  
   *Counts open* syscalls (open, openat, etc.) by process name*

5. **`bpftrace -e 'uprobe:/lib/libc.so.6:malloc { printf("malloc(%d) by %s\n", arg0, comm); }'`**  
   *Traces malloc calls with size and process name*

6. **`bpftrace -e 'uprobe:/lib/libc.so.6:malloc /comm != "bpftrace"/ { printf("%s: malloc(%d)\n", comm, arg0); }'`**  
   *Same as above but excludes bpftrace's own malloc calls*

7. **`bpftrace -e 'uprobe:/lib/libc.so.6:malloc /comm != "bpftrace" && comm != "tmux: server" && comm != "sshd-session"/ { printf("%s: malloc(%d)\n", comm, arg0); }'`**  
   *Extended version excluding more noisy processes*

8. **`bpftrace -lv 'tracepoint:syscalls:sys_enter_openat'`**  
   *Lists available variables for the openat tracepoint*

9. **`bpftrace -lv 'tracepoint:syscalls:sys_exit_openat'`**  
   *Lists available variables for openat exit*

10. **`bpftrace -e 'tracepoint:syscalls:sys_exit_openat /args->ret < 0/ { @[comm] = count(); }'`**  
    *Counts failed openat calls by process name (ret < 0 indicates error)*

### 10 Additional Useful bpftrace One-liners:

1. **Count syscalls by process**  
   ```bash
   bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count(); }'
   ```

2. **Measure syscall latency**  
   ```bash
   bpftrace -e 'tracepoint:syscalls:sys_enter_openat { @start[tid] = nsecs; } 
                tracepoint:syscalls:sys_exit_openat /@start[tid]/ { 
                    @times = hist(nsecs - @start[tid]); delete(@start[tid]); }'
   ```

3. **Track process execution**  
   ```bash
   bpftrace -e 'tracepoint:sched:sched_process_exec { printf("%s -> %s\n", comm, args->filename); }'
   ```

4. **Count page faults by process**  
   ```bash
   bpftrace -e 'software:page-faults:1 { @[comm] = count(); }'
   ```

5. **Monitor TCP connections**  
   ```bash
   bpftrace -e 'tracepoint:syscalls:sys_enter_connect { 
       printf("%s -> %s:%d\n", comm, str(args->uservaddr->sin_addr), args->uservaddr->sin_port); }'
   ```

6. **Track file I/O size distribution**  
   ```bash
   bpftrace -e 'tracepoint:syscalls:sys_exit_read /args->ret > 0/ { 
       @bytes = hist(args->ret); }'
   ```

7. **Monitor process forks**  
   ```bash
   bpftrace -e 'tracepoint:sched:sched_process_fork { 
       printf("%s (PID %d) forked %d\n", args->parent_comm, args->parent_pid, args->child_pid); }'
   ```

8. **Count context switches by process**  
   ```bash
   bpftrace -e 'tracepoint:sched:sched_switch { @[comm] = count(); }'
   ```

9. **Track disk I/O latency**  
   ```bash
   bpftrace -e 'tracepoint:block:block_rq_issue { @start[args->dev] = nsecs; }
                tracepoint:block:block_rq_complete /@start[args->dev]/ {
                    @usecs = hist((nsecs - @start[args->dev]) / 1000);
                    delete(@start[args->dev]); }'
   ```

10. **Monitor memory allocations by size**  
    ```bash
    bpftrace -e 'uprobe:/lib/libc.so.6:malloc { @sizes = hist(arg0); }'
    ```
