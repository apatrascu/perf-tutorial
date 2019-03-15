# Matrix multiplications

First we need to compile the program:

```bash
make clean
make
```

Running the program with `time` gives us:

```bash
time ./matrix

real	0m4.227s
user	0m4.227s
sys	0m0.000s
```

or:

```bash
sudo perf stat -e cpu-clock ./matrix

 Performance counter stats for './matrix':

       4250.018685      cpu-clock (msec)          #    1.000 CPUs utilized          

       4.250402198 seconds time elapsed

```

A bit more details:

```bash
sudo perf stat ./matrix

 Performance counter stats for './matrix':

       4237.217880      task-clock (msec)         #    1.000 CPUs utilized          
                 3      context-switches          #    0.001 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
             2,971      page-faults               #    0.701 K/sec                  
     9,745,549,723      cycles                    #    2.300 GHz                    
    21,161,441,170      instructions              #    2.17  insn per cycle         
     1,036,319,184      branches                  #  244.575 M/sec                  
         1,109,088      branch-misses             #    0.11% of all branches        

       4.237634969 seconds time elapsed
```


Now let's profile it:

```bash
sudo perf record ./matrix
[ perf record: Woken up 3 times to write data ]
[ perf record: Captured and wrote 0.673 MB perf.data (17251 samples) ]
```

Let's see the results:

```bash
sudo perf report
```

You should see something like:

![1](img/Selection_009.png?raw=true)

Or, if you prefer not to use the ncurses ui:

```bash
sudo perf report --stdio --sort comm,dso
# To display the perf.data header info, please use --header/--header-only options.
#
#
# Total Lost Samples: 0
#
# Samples: 17K of event 'cycles:ppp'
# Event count (approx.): 9845467764
#
# Overhead  Command  Shared Object    
# ........  .......  .................
#
    99.51%  matrix   matrix           
     0.34%  matrix   libc-2.23.so     
     0.15%  matrix   [kernel.kallsyms]
     0.00%  perf     [kernel.kallsyms]
```

The `--sort` option aggregates ("sums up") and sorts the results by command (`comm`), dynamically shared object (`dso`), process identifier (`pid`), and more. The `--sort comm,dso` option aggregates the results by command and shared object (i.e., binary executable module)

We can see that we spend most of our time in the `multiply_matrices` function. 

Let's try a simple optimization for this function:

```C
void multiply_matrices() {
    int i, j, k ;
    for (j=0; j<MSIZE; j++) {
        for (i=0; i<MSIZE; i++) {
            float sum = 0.0 ;
            for (k=0;  k<MSIZE; k++) {
                sum = sum + (matrix_a[i][k] * matrix_b[k][j]) ;
            }
            matrix_r[i][j] = sum ;
        }
    }
}
```

If we modify the `multiply_matrices` as above (swap the `i` and `j` loops) and run the perf tool again, you will get something like:

```bash
sudo perf stat ./matrix

 Performance counter stats for './matrix':

       4086.177384      task-clock (msec)         #    1.000 CPUs utilized          
                 3      context-switches          #    0.001 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
             2,971      page-faults               #    0.727 K/sec                  
     9,398,169,202      cycles                    #    2.300 GHz                    
    21,161,493,943      instructions              #    2.25  insn per cycle         
     1,036,333,820      branches                  #  253.619 M/sec                  
         1,108,763      branch-misses             #    0.11% of all branches        

       4.086579285 seconds time elapsed
```

We can observe that the number of cycles are reduced from 9,745,549,723 to 9,398,169,202, which means a 4% improvement just by this simple trick.

Other solutions are up to you.
