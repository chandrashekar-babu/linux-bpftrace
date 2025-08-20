## CPU Profiling and Stack Analysis

```bash
# Profile specific process (8582) at 99Hz, capturing kernel and user stacks
bpftrace -e 'profile:hz:99 /pid == 8582/ { @[kstack, ustack] = count(); } interval:s:10 { exit(); }'

# Various profiling attempts with different intervals and PID filters
bpftrace -e 'profile:hz:99 /pid == 8582/ { @[kstack, ustack] = count(); }'
bpftrace -e 'interval:ms:500 /pid == 8582/ { @[kstack, ustack] = count(); }'
bpftrace -e 'interval:ms:500 /pid == 8582/ { @[kstack] = count(); }'
bpftrace -e 'interval:ms:500 /pid == 8660/ { @[kstack] = count(); }'

# Profile with different timing mechanisms and output formats
bpftrace -e 'interval:ms:500 /pid == 8660/ { @kstack[pid] = count(); } END { print(@kstack); }'
bpftrace -e 'profile:ms:100 /pid == 8660/{ @kstack[pid] = count(); } END { print(@kstack); }'
bpftrace -e 'profile:hz:10 /pid == 8660/ { @kstack[pid] = count(); } END { print(@kstack); }'

# General system-wide profiling
bpftrace -e 'profile:hz:20 { @[kstack] = count(); }'
bpftrace -e 'profile:hz:20 { @[func] = count(); }'
bpftrace -e 'profile:hz:20 /pid == 8582/ { @[func] = count(); }'
bpftrace -e 'profile:hz:20 { @[comm] = count(); }'
```

## Custom Script Development

```bash
# Running custom stack tracking script for different PIDs
bpftrace sample_stack_track.bt
bpftrace sample_stack_track.bt 8582
bpftrace sample_stack_track.bt 1523
bpftrace sample_stack_track.bt 9939

# Custom CPU profiling script
./cpu_profiling1.bt
```

## Scheduler and CPU Analysis

```bash
# Examine scheduler switch tracepoint format
bpftrace -lv tracepoint:sched:sched_switch

# Trace process context switches with timestamps
bpftrace -e 'tracepoint:sched:sched_switch { printf("%s: %s[%d] --> %s[%d]\n", time("%H:%M:%S"), args->prev_comm, args->prev_pid, args->next_comm, args->next_pid); }'
bpftrace -e 'tracepoint:sched:sched_switch { printf("%u: %s[%d] --> %s[%d]\n", nsecs, args->prev_comm, args->prev_pid, args->next_comm, args->next_pid); }'

# Off-CPU analysis script for specific processes
./offcpu.bt 8660
./offcpu.bt 9939

# Set CPU affinity for processes before analysis
taskset -pc 4 8582
taskset -pc 4 8660
taskset -pc 4 9939
```

## Built-in Tool Exploration

```bash
# Explore bpftrace and BCC tool locations
pacman -Ql bpftrace
cd /usr/share/bpftrace/tools/
cd /usr/share/bcc/tools/

# Run standard bpftrace tools
./runqlat.bt
./runqlen.bt

# Examine BCC tools
vi runqlat
vi vfsstat
./vfsstat
./profile
```

## Memory Allocation Analysis

```bash
# Trace kernel memory allocation sizes by command
bpftrace -e 'kprobe:kmalloc { @bytes[comm] = hist(arg1); }'

# Trace kmalloc using tracepoint (more reliable)
bpftrace -lv tracepoint:kmem:kmalloc
bpftrace -e 'tracepoint:kmem:kmalloc { @bytes[comm] = hist(args->bytes_req); }'

# Trace user-space malloc calls
bpftrace -e 'uprobe:libc:malloc { @bytes[comm] = hist(arg0); }'

# Custom memory analysis scripts
./kmalloc_internal_fragmentation.bt
./memleak_detect.bt
./heap_analyze.bt 8582
./heap_analyze.bt 13632
./heap_analyze.bt 13866
```

## Memory Management and Reclaim Tracing

```bash
# Explore memory reclaim tracepoints
bpftrace -lv kprobe:direct_reclaim_begin
bpftrace -lv '*direct_reclaim*'
bpftrace -l '*direct_reclaim*'
bpftrace -lv tracepoint:vmscan:mm_vmscan_direct_reclaim_begin

# Trace direct memory reclaim events
bpftrace -e 'tracepoint:vmscan:mm_vmscan_direct_reclaim_begin { printf("Memory reclaim initiated at %d\n", nsecs / 1000000); @reclaims[kstack] = count(); }'

# Explore memory compaction
bpftrace -l "*compact_zone*"
ps ax | grep kcompactd
```

## I/O and Block Layer Analysis

```bash
# Explore block I/O tracepoints
bpftrace -l 'tracepoint:block:block_rq_issue'
bpftrace -lv 'tracepoint:block:block_rq_issue'

# Run bitesize analysis script
./bitesize.bt

# Generate I/O workload for testing
sysctl vm.drop_caches=3
ls -lR /usr
find /
```

## Scheduler Function Tracing

```bash
# Explore scheduler functions
bpftrace -l kprobe:pick_next_task_fair
bpftrace '*pick_next_task*'
```

## Using perf Tool for Instrumenting Scheduler

```bash
# Perf scheduler analysis on specific CPU
perf sched record -C 4 sleep 10
perf sched latency
perf sched map
perf sched timehist
perf timechart
```

