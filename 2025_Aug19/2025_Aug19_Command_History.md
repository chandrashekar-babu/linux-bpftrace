I
## Hardware Performance Monitoring

```bash
# Count cache misses per process ID (with 1e6 sampling rate)
bpftrace -e 'hardware:cache-misses:1e6 { @[pid] = count(); }'

# Count cache misses per process ID (default sampling)
bpftrace -e 'hardware:cache-misses { @[pid] = count(); }'

# Count CPU instructions executed per command
bpftrace -e 'hardware:instructions { @[comm] = count(); }'

# Count CPU instructions with separate major/minor counters (incorrect syntax)
bpftrace -e 'hardware:instructions { @major[comm] = count(); @minor[comm] = count() }'
```

## Software Events (Page Faults)

```bash
# Count page faults by command name
bpftrace -e 'software:faults { @[comm] = count(); }'

# Count major and minor page faults separately
bpftrace -e 'software:major-faults { @major[comm] = count(); } software:minor-faults { @minor[comm] = count(); }'

# Count major page faults specifically for python processes
bpftrace -e 'software:major-faults /comm == "python"/ { @[comm] = count(); }'

# Count context switches by command
bpftrace -e 'software:cs { @[comm] = count(); }'
```

## Interval-based Sampling

```bash
# Print and clear syscall counts every second (various interval syntax attempts)
bpftrace -e 'interval:1s { print(@syscalls); clear(@syscalls); }'
bpftrace -e 'interval:us:1s { print(@syscalls); clear(@syscalls); }'
bpftrace -e 'interval:us:1 { print(@syscalls); clear(@syscalls); }'

# Simple interval printing examples
bpftrace -e 'interval:1s { print("Ticking..."); }'
bpftrace -e 'interval:s:1 { print("Ticking..."); }'
bpftrace -e 'interval:ms:500 { print("Ticking..."); }'
bpftrace -e 'interval:hz:1 { print("Ticking..."); }'
bpftrace -e 'interval:hz:10 { print("Ticking..."); }'
bpftrace -e 'profile:hz:1 { print("Ticking..."); }'
```

## Probe Discovery and Listing

```bash
# List all hardware events
bpftrace -l "hardware:*"

# List all software events
bpftrace -l "software:*"

# List tracepoints (counting and examining)
bpftrace -l "tracepoint:*" | wc -l
bpftrace -l "tracepoint:sched:*" | wc -l
bpftrace -l "tracepoint:sched:*"

# List specific probes with wildcards
bpftrace -l "*vfs_open*"
bpftrace -l "*kfree*"

# List with verbose output for specific probes
bpftrace -lv kprobe:vfs_open
bpftrace -lv tracepoint:syscalls:sys_enter_openat
bpftrace -lv tracepoint:syscalls:sys_enter_execve
bpftrace -lv fentry:vfs_open
bpftrace -lv fexit:vfs_open
```

## System Call Tracing

```bash
# Trace open syscalls by program and PID
bpftrace -e 'tracepoint:vfs:vfs_read { @program[comm,pid] = count(); }'

# Raw tracepoint for syscall exit (return value tracing)
bpftrace -e 'rawtracepoint:sys_exit { print(args->ret); }'
bpftrace -e 'rawtracepoint:sys_exit { print((int64)arg0); }'
```

## Network Tracing

```bash
# Count TCP reset events
bpftrace -e 'kprobe:tcp_reset { @tcp_resets = count(); }'

# Count TCP receive events
bpftrace -e 'kprobe:tcp_v4_rcv { @tcp_rcvs = count(); }'
```

## Memory Allocation Tracing

```bash
# Trace libc malloc calls and show allocation size
bpftrace -e 'uprobe:libc:malloc { printf("%s allocated %u bytes.\n", comm, arg0); }'

# Exclude bpftrace itself from malloc tracing
bpftrace -e 'uprobe:libc:malloc  /comm != "bpftrace"/{ printf("%s allocated %u bytes.\n", comm, arg0); }'

# Trace kernel memory allocation
bpftrace -lv 'tracepoint:kmem:kmalloc'
bpftrace -lv 'tracepoint:kmem:kfree'
```

## Process and Task Analysis

```bash
# Iterate through all tasks and print PID/command
bpftrace -e 'iter:task { printf("%6d %-20s\n", ctx->task->pid, ctx->task->comm); }'

# Enhanced task iteration with PPID and UID
bpftrace -e 'iter:task { printf("%6d %6d %-20s\n", ctx->task->pid, ctx->task->ppid, ctx->task->comm); }'
bpftrace -e 'iter:task { printf("%6d %6d %-20s\n", ctx->task->pid, ctx->task->uid, ctx->task->comm); }'

# Analyze virtual memory areas for processes
bpftrace -e 'iter:task_vma { printf("%10s: %x - %x\n", ctx->task->comm, ctx->vma->vm_start, ctx->vma->vm_end); }'

# Filter VMA analysis for specific processes
bpftrace -e 'iter:task_vma /comm == "bash"/ { printf("%10s: %x - %x\n", ctx->task->comm, ctx->vma->vm_start, ctx->vma->vm_end); }'
```

## Kernel Function Tracing

```bash
# Trace vfs_open function entry with stack traces
bpftrace -e 'fentry:vfs_open { kstack(); }'
bpftrace -e 'fentry:vfs_open { @[comm] = kstack(); }'

# Attempt to trace vfs_open with path extraction (complex structure access)
bpftrace -e 'kprobe:vfs_open { printf("open path: %s\n", str(((struct path *)arg0)->dentry->d_name.name)); }'
```

## Stack Tracing

```bash
# Capture kernel stacks on vfs_open
bpftrace -e 'fentry:vfs_open { @[comm] = kstack(); }'

# Capture user stacks on malloc calls
bpftrace -e 'uprobe:libc:malloc { @[comm] = ustack(); }'
```

## Watchpoints (Hardware Breakpoints)

```bash
# Set watchpoint on kernel memory address for read/write operations
bpftrace -e 'watchpoint:0xffffffff826326e8:32:r { print(kstack()); }'
bpftrace -e 'watchpoint:0xffffffff826326e8:8:r { print(kstack()); }'
bpftrace -e 'watchpoint:0xffffffff826326e8:8:rw { print(kstack()); }'
```

## Language Features and Testing

```bash
# Variable testing (scalar vs map variables)
bpftrace -e 'BEGIN { $a = 10; printf("%d\n", $a); }'
bpftrace -e 'BEGIN { @a = 10; printf("%d\n", @a); }'

# Tuple testing
bpftrace -e 'BEGIN { $a = (10, 15, 20); printf("%d - %d\n", $a.0, $a.2); }'
bpftrace -e 'BEGIN { $a = (10, "testdata", 20); printf("%d - %s\n", $a.0, $a.1); }'

# Time unit testing
bpftrace -e 'BEGIN { $a = 1m; printf("%d\n", $a); }'
bpftrace -e 'BEGIN { $a = 1s; printf("%d\n", $a); }'

# Loop constructs testing
bpftrace -e 'BEGIN { for ($i: 1..10) { printf("Counting %d\n", $i); } }'
bpftrace -e 'BEGIN { for ($i : 0..ncpus) { printf("Counting %d\n", $i); } }'

# CPU affinity testing
taskset -c 4 bpftrace -e 'BEGIN { print(cpu); }'

# CPU usage per core
bpftrace -e 'tracepoint:sched:sched_switch { @cpus[cpu] = count(); }'
```

## File-based Scripts
```bash
# Running bpftrace scripts from files
bpftrace 02_for_loop_test.bt
bpftrace while_loop_test.bt
./count_bytes_read.bt 9709
```
