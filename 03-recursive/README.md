# Recursive functions

First we need to compile the program:

```bash
make clean
make
```

Running the program with `time` gives us:

```bash
time ./fib

real    0m5.078s
user    0m5.074s
sys 0m0.004s
```

Now let's profile it:

```bash
sudo perf record ./fib
[ perf record: Woken up 4 times to write data ]
[ perf record: Captured and wrote 0.802 MB perf.data (20634 samples) ]
```

Let's see the results:

```bash
sudo perf report
```

You should see something like:

![1](img/Selection_001.png?raw=true)

Okay, great. So basically, it doesn't tell us too much. It does say that we're spending all of our time in the `fib` function, but not much else. 

We can hit enter with the `fib` function highlighted, we get a few options.
 
![2](img/Selection_002.png?raw=true)

The first one is to annotate the function. This shows us a disassembly of the instructions in the function, with the percentage of time being taken by each one. 

![3](img/Selection_003.png?raw=true)

By default, perf only collects time information. We probably want to see a callgraph since we're making calls.

To get the call graph, we pass the `-g` option:

```bash
sudo perf record -g ./fib
```

And wait for it to run again... Then run `perf report` again to see the new view.

![4](img/Selection_006.png?raw=true)

Not super different. Neat! Okay, so there are some differences between the first run and the second. Namely, instead of having an overhead column, it now has a `Children` and `Self` column, as well as some `+` signs next to some function calls.

`Children` refers to the overhead of subsequent calls, while `Self` refers to the cost of the function itself. A high children overhead, and a low self overhead indicates that the function makes a call that is expensive, but a high self overhead indicates that the function itself is expensive.

The `+` indicates that the function makes a call that is accessible. Not all functions are follow-able, likely due to the optimizer striping the function names to shrink the binary.

Highlighting a row and hitting enter will expand it, showing the functions called. 

![5](img/Selection_007.png?raw=true)

Let's do it for all `fib` calls.

![6](img/Selection_008.png?raw=true)

Going back to the disassembly, most of the time is spent doing the `mov %rsp,%rbp` which are the registers responsible for the stack and frame pointers in the x86 architecture. 

Since we see that there is a chain of calls to the fib function, and this particular instruction is quite hot, there's a good indication that something about the recursion is to blame. 

Now we know what to fix. Fixing it is up to you. Some solutions might include memoization, or emulating recursion with a stack and a while loop, or even a combination of both.


