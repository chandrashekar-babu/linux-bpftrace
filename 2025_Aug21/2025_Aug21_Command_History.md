
## Network Analysis Script Development

```bash
# TCP connection and retransmission analysis scripts
bpftrace 05_tcp_retransmit_skb.bt
vi 05_tcp_retransmit_skb.bt  # Multiple edits and test runs

# DNS monitoring scripts
cp ../2025_Aug20/dns_lookup_analyze.bt .
vi dns_lookup_analyze.bt
vi dns_query_monitoring.bt
bpftrace dns_query_monitoring.bt

# Various network analysis scripts created
vi http_request_header_monitoring.bt
vi nic_queue_monitoring.bt
vi tcp_receive_window_analysis.bt
vi network_driver_io.bt
vi sock_open_close.bt
vi sock_connection_tracking.bt
vi unauthorized_net_activity.bt
vi netdata_exfiltration.bt
```

## Block I/O Analysis

```bash
# Trace block request issues with sector information
bpftrace -e 'tracepoint:block:block_rq_issue { printf("%s %s %s %d sectors\n", probe, args->comm, args->rwbs, args->nr_sector); }'

# Examine block tracepoint format
bpftrace -lv tracepoint:block:block_rq_issue

# Custom block I/O performance script
vi block_io_performance.bt
bpftrace block_io_performance.bt

# Use standard bpftrace I/O tools
./bitesize.bt
less bitesize.bt
./biolatency.bt
vi biostacks.bt
vi biosnoop.bt
./biosnoop.bt
```

## Socket and Network Tracing

```bash
# Explore socket-related tracepoints
bpftrace -l 'tracepoint:sock:*'
bpftrace -lv 'tracepoint:sock:sock_rcvqueue_full'

# Trace socket receive queue full events
bpftrace -e 'tracepoint:sock:sock_rcvqueue_full { printf("rmem_alloc=%d, trusize=%u, sk_rcvbuf=%d\n", args->rmem_alloc, args->truesize, args->sk_rcvbuf); }'

# Trace socket data ready events with stack
bpftrace -e 'tracepoint:sock:sk_data_ready { print(kstack); }'
```

## TCP Connection Analysis

```bash
# Explore TCP connection syscalls
bpftrace -l 'tracepoint:syscalls:sys_enter_connect*'
bpftrace -lv 'tracepoint:syscalls:sys_enter_connect'

# Various connect tracing attempts
bpftrace -e 'uprobe:libc:connect { printf("%s: addr=0x%llx\n", comm, arg0); }'
bpftrace -e 'uprobe:libc:connect { printf("%s: addr=0x%llx\n", comm, arg1); }'
bpftrace -e 'tracepoint:syscalls:sys_enter_connect { printf("%s: addr=0x%llx\n", comm, args->sockaddr); }'
bpftrace -e 'tracepoint:syscalls:sys_enter_connect { printf("%s: addr=0x%llx\n", comm, args->uservaddr); }'

# Use standard tcpconnect tool
./tcpconnect.bt
vi tcpconnect.bt
```

## Network Device Tracing

```bash
# Explore network tracepoints
bpftrace -l tracepoint:tcp:*
bpftrace -l tracepoint:net:*
bpftrace -lv tracepoint:net:net_dev_start_xmit

# Trace network device transmission
bpftrace -e 'tracepoint:net:net_dev_start_xmit { printf("%s: %u bytes\n", args->name, args->data_len); }'
bpftrace -e 'tracepoint:net:net_dev_start_xmit { printf("%s: %u bytes\n", str(args->name), args->data_len); }'
bpftrace -e 'tracepoint:net:net_dev_start_xmit { printf("%s: %u bytes\n", comm, args->data_len); }'
bpftrace -e 'tracepoint:net:net_dev_start_xmit { @sent[comm] += args->data_len; }'

# Trace network transmission with stack analysis
bpftrace -e 'tracepoint:net:net_dev_start_xmit { print(kstack); }'

# Explore network receive tracepoints
bpftrace -lv tracepoint:net:net_receive_skb
bpftrace -lv tracepoint:net:netif_receive_skb

# Trace network receive events
bpftrace -e 'tracepoint:net:netif_receive_skb { printf("%s: %u bytes\n", args->name, args->len); }'
bpftrace -e 'tracepoint:net:netif_receive_skb { print(kstack); }'
```

## Advanced TCP Analysis Scripts

```bash
# TCP connection tracking script
vi 01_tcp_connection_track.bt
bpftrace 01_tcp_connection_track.bt

# TCP life tracking
vi tcplife.bt
./tcplife.bt

# TCP round-trip time analysis
vi tcp_round_trip_time.bt
bpftrace tcp_round_trip_time.bt  # Multiple test runs and edits

# Explore TCP kernel functions
bpftrace -l kprobe:tcp_trans*
bpftrace -l kprobe:tcp_*
bpftrace -l kprobe:*tcp_transmit*
bpftrace -l kprobe:sock_alloc_send_pskb
```

## Security and Monitoring Scripts

```bash
# Unauthorized connection tracking
vi track_unauthorized_socket_connects.bt
bpftrace track_unauthorized_socket_connects.bt

# DNS lookup analysis
bpftrace dns_lookup_analyze.bt
```

## System Exploration and Testing

```bash
# Generate network traffic for testing
curl -o /tmp/test.dat https://www.kernel.org/
curl -O https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.16.2.tar.xz
links www.kernel.org
elinks www.kernel.org

# System exploration
cat /proc/sys/net/ipv4/tcp_syncookies
man getaddrinfo
man getaddrinfo_a
man gethostbyname

# Generate I/O load for testing
sysctl vm.drop_caches=3
find /
cat /boot/vmlinuz-linux-lts > /dev/null
```

