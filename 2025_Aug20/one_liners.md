# Show top kernel functions consuming CPU
bpftrace -e 'profile:hz:99 { @[kstack] = count(); }'

# Sample process 1234 and print top user functions
bpftrace -e 'profile:hz:99 /pid == 1234/ { @[func] = count(); }'

# Identify CPU time by process name
bpftrace -e 'profile:hz:99 { @[comm] = count(); }'

# Capture stacks for flame graph
bpftrace -e 'profile:hz:99 /pid == 1234/ { @[ustack] = count(); }' > stacks.out

# Generate flame graph
cat stacks.out | flamegraph.pl > profile.svg

# Track time spent off-CPU for process 1234
bpftrace -e '
tracepoint:sched:sched_switch / args->prev_pid == 1234 / {
  @start[args->prev_pid] = nsecs;
}

tracepoint:sched:sched_switch / args->next_pid == 1234 / {
  $pid = args->next_pid;
  $ts = nsecs;
  $start_ts = @start[$pid];
  if ($start_ts != 0) {
    @off_cpu_time[$pid, comm] = sum($ts - $start_ts);
    delete(@start[$pid]);
  }
}
'

#### Memory Analysis
# Trace all kmalloc calls by size
bpftrace -e '
kprobe:kmalloc {
  @bytes[comm] = hist(arg1);
}
'

# Track malloc() calls by size
bpftrace -e '
uprobe:/lib/x86_64-linux-gnu/libc.so.6:malloc {
  @bytes[comm] = hist(arg0);
}
'
# Track major page faults by process
bpftrace -e '
software:major-faults {
  @faults[comm, pid] = count();
}
'

# Track minor page faults 
bpftrace -e '
software:minor-faults {
  @faults[comm, pid] = count();
}
'

# Monitor when system starts reclaiming memory
bpftrace -e '
kprobe:direct_reclaim_begin {
  printf("Memory pressure: direct reclaim started at %u\n", nsecs/1000000);
  @reclaims[kstack] = count();
}
'

# Monitor memory compaction events
bpftrace -e '
kprobe:compact_zone {
  printf("Memory compaction event at %u\n", nsecs/1000000);
  @compactions[kstack] = count();
}
'

#### Scheduler latency 



##### Lab
# Terminal 1: Create artificial CPU load
for i in $(seq 1 $(nproc)); do 
  stress-ng --cpu 1 --timeout 60s &
done

# Terminal 2: Create a sensitive test process
while true; do 
  read -t 0.001 || echo -n "."; 
  sleep 0.01; 
done

### Block I/O Latency
# Trace block I/O request start
bpftrace -e '
tracepoint:block:block_rq_issue {
 printf("%s %s %s %d sectors\n", 
 probe, args->comm, args->rwbs, args->nr_sector);
}
'

# Monitor I/O latency
bpftrace -e '
BEGIN {
 printf("Tracing block I/O... Hit Ctrl-C to end.\n");
}

tracepoint:block:block_rq_issue {
 // Start time for I/O request
 @start[args->dev, args->sector] = nsecs;
 @type[args->dev, args->sector] = args->rwbs;
}

tracepoint:block:block_rq_complete {
 $start = @start[args->dev, args->sector];
 $type = @type[args->dev, args->sector];
 
 if ($start) {
 $duration = nsecs - $start;
 // Create histogram of I/O latency by type (read/write)
 @usecs[$type] = hist($duration / 1000);
 delete(@start[args->dev, args->sector]);
 delete(@type[args->dev, args->sector]);
 }
}
'

#### Filesystem I/O Analysis

# Monitor file opens with path info
bpftrace -e '
tracepoint:syscalls:sys_enter_open,
tracepoint:syscalls:sys_enter_openat {
  @files[comm, str(args->filename)] = count();
}
'

# Track read/write sizes by file
bpftrace -e '
tracepoint:syscalls:sys_exit_read /args->ret > 0/ {
  @reads[comm, pid] = hist(args->ret);
}

tracepoint:syscalls:sys_exit_write /args->ret > 0/ {
  @writes[comm, pid] = hist(args->ret);
}
'

# Detect direct synchronous writes
bpftrace -e '
kprobe:sync_page {
  @sync_by_proc[comm] = count();
}

kprobe:fsync {
  @fsync_by_proc[comm] = count();
  @fsync_stacks[ustack, comm] = count();
}
'

#### Network Performance Analysis

# Monitor socket connections
bpftrace -e '
tracepoint:syscalls:sys_enter_connect {
  @connects[comm] = count();
}
'

# Track TCP retransmits
bpftrace -e '
kprobe:tcp_retransmit_skb {
  @retransmits[comm] = count();
}
'

