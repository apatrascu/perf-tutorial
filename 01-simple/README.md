# Simple perf usage

## To obtain a list of supported events:

```bash
perf list

List of pre-defined events (to be used in -e):

  branch-instructions OR cpu/branch-instructions/    [Kernel PMU event]
  branch-misses OR cpu/branch-misses/                [Kernel PMU event]
  bus-cycles OR cpu/bus-cycles/                      [Kernel PMU event]
  cache-misses OR cpu/cache-misses/                  [Kernel PMU event]
  cache-references OR cpu/cache-references/          [Kernel PMU event]
  cpu-cycles OR cpu/cpu-cycles/                      [Kernel PMU event]
  cstate_core/c3-residency/                          [Kernel PMU event]
  cstate_core/c6-residency/                          [Kernel PMU event]
  cstate_core/c7-residency/                          [Kernel PMU event]
  cstate_pkg/c2-residency/                           [Kernel PMU event]
  cstate_pkg/c3-residency/                           [Kernel PMU event]
  cstate_pkg/c6-residency/                           [Kernel PMU event]
  cstate_pkg/c7-residency/                           [Kernel PMU event]
  instructions OR cpu/instructions/                  [Kernel PMU event]
  mem-loads OR cpu/mem-loads/                        [Kernel PMU event]
  mem-stores OR cpu/mem-stores/                      [Kernel PMU event]
  msr/aperf/                                         [Kernel PMU event]
  msr/mperf/                                         [Kernel PMU event]
  msr/smi/                                           [Kernel PMU event]
  msr/tsc/                                           [Kernel PMU event]
  power/energy-cores/                                [Kernel PMU event]
  power/energy-pkg/                                  [Kernel PMU event]
  power/energy-ram/                                  [Kernel PMU event]
  ref-cycles OR cpu/ref-cycles/                      [Kernel PMU event]
  topdown-fetch-bubbles OR cpu/topdown-fetch-bubbles/ [Kernel PMU event]
  topdown-recovery-bubbles OR cpu/topdown-recovery-bubbles/ [Kernel PMU event]
  topdown-slots-issued OR cpu/topdown-slots-issued/  [Kernel PMU event]
  topdown-slots-retired OR cpu/topdown-slots-retired/ [Kernel PMU event]
  topdown-total-slots OR cpu/topdown-total-slots/    [Kernel PMU event]
  uncore_imc_0/cas_count_read/                       [Kernel PMU event]
  uncore_imc_0/cas_count_write/                      [Kernel PMU event]
[...]
```

## Counting events with `perf stat`

### Simple usage
For any of the supported events, `perf` can keep a running count during process execution. In counting modes, the occurrences of events are simply aggregated and presented on standard output at the end of an application run. 

To generate these statistics, use the stat command of perf. For instance:

```bash
sudo perf stat -B dd if=/dev/zero of=/dev/null count=10000001000000+0 records in

1000000+0 records out
512000000 bytes (512 MB, 488 MiB) copied, 0.995514 s, 514 MB/s

 Performance counter stats for 'dd if=/dev/zero of=/dev/null count=1000000':

        995.943403      task-clock (msec)         #    1.000 CPUs utilized          
                 1      context-switches          #    0.001 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
                68      page-faults               #    0.068 K/sec                  
     2,290,650,065      cycles                    #    2.300 GHz                    
     2,329,116,265      instructions              #    1.02  insn per cycle         
       472,390,465      branches                  #  474.315 M/sec                  
         9,016,754      branch-misses             #    1.91% of all branches        

       0.996273757 seconds time elapsed
```

With no events specified, perf stat collects the common events listed above. Some are software events, such as context-switches, others are generic hardware events such as cycles. After the hash sign, derived metrics may be presented, such as 'IPC' (instructions per cycle).

**By default, events are measured at both user and kernel levels!**

### Filtering events

If you want just the userspace events:

```bash
sudo perf stat -e cycles:u dd if=/dev/zero of=/dev/null count=100000
100000+0 records in
100000+0 records out
51200000 bytes (51 MB, 49 MiB) copied, 0.0999541 s, 512 MB/s

 Performance counter stats for 'dd if=/dev/zero of=/dev/null count=100000':

        22,163,381      cycles:u                                                    

       0.100687052 seconds time elapsed
```

If you want just the kernelspace events:

```bash
sudo perf stat -e cycles:k dd if=/dev/zero of=/dev/null count=100000
100000+0 records in
100000+0 records out
51200000 bytes (51 MB, 49 MiB) copied, 0.100322 s, 510 MB/s

 Performance counter stats for 'dd if=/dev/zero of=/dev/null count=100000':

       209,501,880      cycles:k                                                    

       0.101023943 seconds time elapsed
```

### Repeated measurements

```bash
sudo perf stat -r 5 sleep 1

 Performance counter stats for 'sleep 1' (5 runs):

          0.461651      task-clock (msec)         #    0.000 CPUs utilized            ( +-  3.67% )
                 1      context-switches          #    0.002 M/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
                58      page-faults               #    0.126 M/sec                  
         1,014,481      cycles                    #    2.198 GHz                      ( +-  1.13% )
           744,921      instructions              #    0.73  insn per cycle           ( +-  0.23% )
           147,858      branches                  #  320.280 M/sec                    ( +-  0.18% )
             6,881      branch-misses             #    4.65% of all branches          ( +-  0.45% )

       1.000868758 seconds time elapsed                                          ( +-  0.00% )
```

### Processor-wide mode

By default, perf stat counts in per-thread mode. To count on a per-cpu basis pass the `-a` option. When it is specified by itself, all online processors are monitored and counts are aggregated. For instance:

```bash
udo perf stat -B -ecycles:u,instructions:u -a dd if=/dev/zero of=/dev/null count=2000000
2000000+0 records in
2000000+0 records out
1024000000 bytes (1.0 GB, 977 MiB) copied, 2.00107 s, 512 MB/s

 Performance counter stats for 'system wide':

       998,013,395      cycles:u                                                    
       955,180,502      instructions:u            #    0.96  insn per cycle         

       2.003339816 seconds time elapsed

```

It is possible to restrict monitoring to a subset of the CPUS using the -C option. A list of CPUs to monitor can be passed. For instance, to measure on CPU0, CPU2 and CPU3:

```bash
sudo perf stat -B -e cycles:u,instructions:u -a -C 0,2-3 sleep 5

 Performance counter stats for 'system wide':

        59,939,564      cycles:u                                                    
        28,308,431      instructions:u            #    0.47  insn per cycle         

       5.000833871 seconds time elapsed
```

### Attaching to a running process

It is possible to use perf to attach to an already running thread or process. This requires the permission to attach along with the thread or process ID. To attach to a process, the -p option must be the process ID. To attach to the sshd service that is commonly running on many Linux machines, issue:

```bash
ps ax | grep sshd
 2042 ?        Ss     0:00 /usr/sbin/sshd -D
32204 pts/43   S+     0:00 grep --color=auto sshd
```

and then:

```bash
sudo perf stat -e cycles -p 2042 sleep 2

 Performance counter stats for process id '2042':

     <not counted>      cycles                                                      

       2.000768032 seconds time elapsed
```

## Sampling with `perf record`

### Collecting samples

```bash
sudo perf record sleep 5
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.016 MB perf.data (9 samples) ]
```

### Reporting
```bash
sudo perf report
```

## Real time statistics

Display all cycles events:

```bash
sudo perf top -a
```

Display all cpu-clock related events:

```bash
perf top -e cpu-clock
```