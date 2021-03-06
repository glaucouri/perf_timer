![PyPI - Python Version](https://img.shields.io/pypi/pyversions/perf_tool)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/glaucouri/perf_tool)
![Travis (.com)](https://img.shields.io/travis/com/glaucouri/perf_tool)


## perf_tool
Another **time performance** tool for python.
It is the right support tool if you need to profile an entire program, some piece of code, 
a module, a class, a function or a single row. The idea here is to profile only the code **you want** to profile.

### Installation

perf_tool is tested on python 2.7, 3.6, 3.7, 3.8 

```
> pip install perf_tool
```

To run all the tests run *pytest* or tox for *multiversion*:

```
> tox 
or
> pytest
```

### Description
I have often been faced with the problem of performance, on very challanging scope.

In many cases difficult if given by:

* intrinsic complexity of algorithms, in this cases, piece of codes can't be explained easily with a right asymptotic 
representations aka [Big O notation](https://en.wikipedia.org/wiki/Big_O_notation)

* domain not fully known , yes! when the code is not yours, if you don't know exactly how and why the developer has done 
some choices each row of code can be interpreted as an error candidate.
  
* [coupling with data](https://en.wikipedia.org/wiki/Coupling_(computer_programming)), sometimes algorithms is too much
intricate with data, the result is an unreadable spaghetti code.

* use of external framework like Pandas, Scipy, Numpy. In these cases is not simple get a correct idea of where is spent 
computational time.

In these cases [computational analysis](https://en.wikipedia.org/wiki/Computational_complexity) can be misleading,
on the other hands [python profiler](https://docs.python.org/3.7/library/profile.html) is too dispersive so juggling 
with performance tools give to us very counterintuitive results.

### How it works
When you profile a program your first need is to construct a metric that can tell you where the time is spent.
So at the beginning you must put **sentinels** in the entry points, like constructors, methods or functions, in order to create 
main metrics. Gradually you nest other *sentinels* in the most relevant section of codes, increasing the capillarity 
until individual lines of code are identified.


*perf_tool* has 2 sentinel:

```
> from perf_tool import PerfTool, perf_tool
```

* **PerfTool** Is a context manager 
* **perf_tool** Is a function or method decorator

each *Sentinel* cover a piece of code, and for that, records how many time is spent on it. 

Finally we can print a report containing 4 columns:

* task: task label or name, it is set when you put the sentinel, i suggest to use mnemonic label
* aver(s): mean of time spent ove the piece of code.
* sum(s): sum of time spent ove the piece of code.
* count: how many times that sentinel was called
* std: standard deviation

The hierarchy of the *Sentinels*, that can be nested, is mainteined in order to represent the code structure. 

First step is to put root sentinels: 

```
@perf_tool('main')
def main():
  <some code>
```

Second step is to place sentinel in subset of code.

```
@perf_tool('main')
def main():
  with PerfTool('preparation'):
    <some code part 1>

  with PerfTool('calculus'):
    <some code part 2>

  with PerfTool('output'):
    <some code part 3>
```

And so on

```
@perf_tool('main')
def main():
  with PerfTool('preparation'):
    <some code part 1>

  with PerfTool('calculus'):
    <some code part 2>

  with PerfTool('output'):
    <some code part 3a>
    for each row:
    with PerfTool('row write'):
      <some code part 3b>
```

Finally you can enable or disable collector as you need:

The simplest pattern is:

```
from perf_tool import PerfTool, perf_tool
PerfTool.set_enabled()
  <code with sentinels>
PerfTool.show_stats_if_enabled()
```

###### Reset collector, use with notebooks

Remember that collector is a singleton so it continue to collect... forever!
In an interactive environment this can be annoying.

use

> PerfTool.empty()

to reset counters.

###### Conditional use
Sometimes you want to exclude some kind of situation, like reading data from an external source. In this cases is 
totally useless keep track of the execution time.

> PerfTool.set_enabled(False)  # disable collector

> PerfTool.set_enabled()       # enable collecotor


### Examples
After installation you can try to run a little example and see the report:

```
$ curl https://raw.githubusercontent.com/glaucouri/perf_tool/master/example.py > perf_tool_sample.py
$ python perf_tool_sample.py 
================== PerfTool ==================
task                     |aver(s) |sum(s)  |count   |std     
main                     |   0.065|   0.065|       1|   0.000
  +-call                 |   0.050|   0.050|       1|   0.000
    +-body               |   0.050|   0.050|       1|   0.000
  +-row                  |   0.001|   0.014|      10|   0.000

overall                  |    0.04|    0.06|      13|-       
```

Here a production use case.

```
task                     |aver(s) |sum(s)  |count   |std     
Definition set value     |   0.006| 111.786|   17199|   0.013
  +-Comp3 call           |   0.003|   0.043|      13|   0.000
  +-Comp2 call           |   0.009|  37.078|    4056|   0.009
    +-calculate cache    |   0.018|  36.941|    2016|   0.002
      +-aggregate        |   0.012|  23.772|    2016|   0.002
  +-Comp3 call           |   0.083|  30.259|     364|   0.031
    +-aggregate          |   0.091|  16.431|     180|   0.011
    +-agg+pivot          |   0.092|  13.200|     144|   0.011
  +-Comp1 call           |   0.001|   1.607|    2626|   0.002
  +-Comp4 call           |   0.005|   0.061|      13|   0.000
Comp2 set value          |   1.054|  41.113|      39|   1.497
  +-Comp2 compute meas   |   3.163|  41.113|      13|   0.244
    +-Comp2 tmx          |   0.301|  15.675|      52|   0.014
      +-Factory call     |   0.005|   9.377|    1872|   0.001
        +-Comp3 call     |   0.003|   6.372|    1872|   0.001
    +-Comp2 Comp3        |   0.097|   4.650|      48|   0.124
      +-Factory call     |   0.029|   4.220|     144|   0.041
        +-Comp3 call     |   0.028|   3.985|     144|   0.041
          +-aggregate    |   0.086|   3.851|      45|   0.010
    +-Comp2 Comp1        |   0.056|   2.670|      48|   0.008
      +-Factory call     |   0.003|   1.215|     432|   0.002
        +-Comp1 call     |   0.001|   0.522|     432|   0.002
Comp4  set value         |   0.665|  17.300|      26|   0.719
  +-Comp4 compute all    |   1.441|  17.296|      12|   0.037
    +-call transform     |   0.415|  14.935|      36|   0.019
      +-Factory call     |   0.010|  11.894|    1152|   0.009
        +-Comp2 call     |   0.009|  10.519|    1152|   0.009
          +-calculate cac|   0.018|  10.470|     576|   0.003
            +-agg all    |   0.012|   7.000|     576|   0.002
    +-retrieve data      |   0.003|   0.448|     144|   0.000
```


