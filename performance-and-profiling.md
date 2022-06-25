# Performance & Profiling

### Profiling with pprof <a href="#profiling-with-pprof" id="profiling-with-pprof"></a>

1. Query the pprof cpu endpoint on the node host:
   * **CPU**: `curl -X GET localhost:6060/debug/pprof/profile?seconds=<number> > <filename>`
   * **Heap**: `curl -X GET localhost:6060/debug/pprof/heap?seconds=<number> > <filename>`
   * can query from your local machine by substituting localhost with the IP of the node, depending on your network setup. By doing this, can skip step 2.
2. If querying on the node host, SCP the file to yourself: `scp <filename> <user>@<host>:<path>`
   * E.g. `scp <filename> root@143.182.133.71:/home/roman/umee/pprof`
   * ensure that your ISP or firewall is not blocking the file transfer
3. Run a web server and open up a browser`go tool pprof -http=localhost:8080 <filename>`
   * `graphviz` must be installed

#### [#](https://docs.osmosis.zone/developing/osmosis-core/performance.html#memory)Memory <a href="#memory" id="memory"></a>

[**#**](https://docs.osmosis.zone/developing/osmosis-core/performance.html#causes) **Causes**

The following cause memory issues in Go – Creating substrings and subslices. – Wrong use of the defer statement. – Unclosed HTTP response bodies (or unclosed resources in general). – Orphaned hanging go routines. – Global variables.

[**#**](https://docs.osmosis.zone/developing/osmosis-core/performance.html#interpreting-output) **Interpreting Output**

– `inuse_space`: Means pprof is showing the amount of memory allocated and not yet released. – `inuse_objects`: Means pprof is showing the amount of objects allocated and not yet released. – `alloc_space`: Means pprof is showing the amount of memory allocated, regardless if it was released or not. – `alloc_objects`: Means pprof is showing the amount of objects allocated, regardless if they were released or not.

– `flat`: Represents the memory allocated by a function and still held by that function. – `cum`: Represents the memory allocated by a function or any other function that is called down the stack.

#### [#](https://docs.osmosis.zone/developing/osmosis-core/performance.html#useful-links)Useful links <a href="#useful-links" id="useful-links"></a>

* [Pprof Doc(opens new window)](https://pkg.go.dev/net/http/pprof)
* [Graphviz Download(opens new window)](https://graphviz.org/download/)
* [Using SCP(opens new window)](https://linuxize.com/post/how-to-use-scp-command-to-securely-transfer-files/)
* [Advanced Go Profiling Talk (YouTube)(opens new window)](https://www.youtube.com/watch?v=xxDZuPEgbBU)
* [Notes from the talk above(opens new window)](https://github.com/bradfitz/talk-yapc-asia-2015/blob/master/talk.md)
* [Memory Leaking Scenarios(opens new window)](https://go101.org/article/memory-leaking.html)
* [Great blogpost about profiling heap(opens new window)](https://jvns.ca/blog/2017/09/24/profiling-go-with-pprof/)

### Benchmarking <a href="#benchmarking" id="benchmarking"></a>

#### Best practices <a href="#best-practices" id="best-practices"></a>

* Running the benchmarks on an idle machine not running on battery
* Use `-benchmem` to also get stats on allocated objects and space
* Use `benchstat` to compare performance across different git branches
* Adding -run='$^' or -run=- to each go test command to avoid running the tests too

Benchstat sample output for illustration:

```
name                old time/op    new time/op    delta
Decode-4               2.20s ± 0%     1.54s ± 0%   ~     (p=1.000 n=1+1)
```



For benchstat specifically:

* Using higher -count values if the benchmark numbers aren't stable
  * if you don't, your sample size would be too small and `delta` might not be reported (like in example above) because it is not significant enough.
  * if you do, might take longer since you need multiple runs to get a good sample size
  * people recommend 5 as a good enough sample size

Adding -run='$^' or -run=- to each go test command to avoid running the tests too

#### [#](https://docs.osmosis.zone/developing/osmosis-core/performance.html#example)Example <a href="#example" id="example"></a>

Let's assume that we are working on branch `umee/string` and added some performance improvements to `tree.String()`.

As a result, we would like to bench test like in [the following (opens new window)](https://github.com/osmosis-labs/iavl/blob/141d98dba805ca1960160b1ec98c6f243792e25c/nodedb\_test.go#L33-L46)in iavl.

To get a nice bench summary we would follow these steps:

1. Checkout the `master` branch and get the output of the benchmark:

```
git checkout master

go test -benchmem -run=^$ -bench ^BenchmarkTreeString$ -benchmem -count 5 github.com/cosmos/iavl > bench_string_old.txt
```

&#x20;2\. Checkout our `umee/string` branch and get the output of the benchmark:

```
git checkout master

go test -benchmem -run=^$ -bench ^BenchmarkTreeString$ -benchmem -count 5 github.com/cosmos/iavl > bench_string_new.txt
```

&#x20;3\. Compare the two outputs with `benchstat`:

```
benchstat bench_string_old.txt bench_string_new.txt
```

&#x20;4\. Evaluate the output and attach to your PR, if needed

#### Useful links: <a href="#useful-links-2" id="useful-links-2"></a>

* [Benchstat Doc](https://pkg.go.dev/golang.org/x/perf/cmd/benchstat)
* [Tips for Newcomers](https://github.com/golang/go/issues/23471)
