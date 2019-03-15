# Matrix multiplications

First we need to compile the program:

```bash
make clean
make
```

Running the program with `time` gives us:

```bash
time ./matrix 

real	0m0.444s
user	0m0.444s
sys	0m0.000s
```

or:

```bash
sudo perf stat -e cpu-clock ./matrix

 Performance counter stats for './matrix':

        445.399009      cpu-clock (msec)          #    0.999 CPUs utilized          

       0.445724528 seconds time elapsed
```

A bit more details:

```bash
sudo perf stat ./matrix

 Performance counter stats for './matrix':

        437.753641      task-clock (msec)         #    0.999 CPUs utilized          
                 1      context-switches          #    0.002 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
             2,971      page-faults               #    0.007 M/sec                  
     1,006,814,374      cycles                    #    2.300 GHz                    
     2,121,702,725      instructions              #    2.11  insn per cycle         
       282,839,325      branches                  #  646.115 M/sec                  
           331,909      branch-misses             #    0.12% of all branches        

       0.438042916 seconds time elapsed
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
    for (i = 0 ; i < MSIZE ; i++) {
        for (k = 0 ; k < MSIZE ; k++) {
            for (j = 0 ; j < MSIZE ; j++) {
                matrix_r[i][j] = matrix_r[i][j] + (matrix_a[i][k] * matrix_b[k][j]) ;
            }
        }
    }
}
```

If we modify the `multiply_matrices` as above (swap the `k` and `j` loops) and run the perf tool again, you will get something like:

```bash
sudo perf stat ./matrix

 Performance counter stats for './matrix':

        295.846579      task-clock (msec)         #    0.999 CPUs utilized          
                 0      context-switches          #    0.000 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
             2,971      page-faults               #    0.010 M/sec                  
       680,439,662      cycles                    #    2.300 GHz                    
     1,877,085,941      instructions              #    2.76  insn per cycle         
       283,567,071      branches                  #  958.494 M/sec                  
         1,085,652      branch-misses             #    0.38% of all branches        

       0.296123997 seconds time elapsed
```

We can observe that the number of cycles are reduced from 1,006,814,374 to 680,439,662, which means an ~40% improvement just by this simple trick.

Other solutions are up to you.
