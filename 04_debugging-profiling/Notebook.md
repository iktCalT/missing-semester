# Lecture 1: Lecture 4: Debugging and Profiling [![missing semester][MS_ICON_M]](https://missing.csail.mit.edu/2026/debugging-profiling/)

## Debugging

### Printf Debugging and Logging

"*The most effective debugging tool is still careful thought coupled with judicially placed print statements.*"

Logs are long-lived print statements.  

There are structured (e.g. JSON) and unstructured (printing out texts saying what the program is doing) logs.  

Logging levels (like trace, debug, info, warn, error, critical, ...) are used to differentiate the severity of logs.  You can filter logging based on level, for example, in production environment, you may turn off trace and debug logging. While when you are debugging, you may want to keep the debug logs.  (When we are writing codes, if we think that some information needs to be logged, use log instead of print statements so that we don't have re-compile this part in the end).

A lot of third-party applications have logging built in.  The question is just to figure out how to access it—most command-line cools have `-v` or `--verbose` to give you more information. Some even allows you to use `-v -v` or `-vv` to give more verbose output.  For system services, use `journalctl` tool to access the logs.

### Debuggers

Usually, it's annoying to re-compile or re-deploy each time. Or you don't know what to log. This is when **debuggers** come to help.  The most common debuggers are `gdb` and `lldb`, they can be used in any program. But are used more often on compiled (binary) programs like those written in C/C++/Rust/...  For those written in interpretation languages like Python, people prefer to use language-specific debuggers like PDB.

Reverse debugging. `rr` (record replay) has exactly the same interface as `gdb`, but it can record everything the program does.  *So that the program can be executed in reverse order.* [![record replay debugging][YT_ICON]](https://youtu.be/8VYT9TcUmKs?t=478)

#### Heisenbugs

Heisenbugs refers to bugs that when you try to observe them, they are not there. You'll most commonly run into it when you are doing print debugging. Because print debugging will slow down your program because the program will stop and print something on screen. It is this small delay cause the program behave differently, especially for concurrent programs. Debuggers, especially `rr` (because it will cause more delay), also run into this problem.  So, please pay attention to this problem. If `rr` cannot reproduce the bugs, try to use `gdb` or other ways to debug.

#### An example of `rr`

Here is an example of `rr` [![corruption.c, an example of rr][YT_ICON]](https://youtu.be/8VYT9TcUmKs?t=695)

#### Pros and cons of `rr`

Pros: It would be very useful if you combine `rr` with bash scripts (while loops).

Con: `rr` doesn't work very well in virtual machines, because it needs hardware support.  If you modify your code, you need to re-compile and re-record.

### System Call Tracing

Tracing makes it possible to observe what system calls is doing.

#### `strace`

The most commonly used tracing tools are `strace` (Linux) and `dtruss` (macOS). For example, `strace ls -l` will record system calls made by `ls -l`. [![strace example][YT_ICON]](https://youtu.be/8VYT9TcUmKs?t=1538)

`strace` will record only system calls. You can filter which kind of system calls to show. For example, `-e%file` will show only file operations (check the manual page of `stack` for more information).

If one program call another program, `strace` will not record system calls called by the child program. But `-f` or `--follow-forks` will trace all system calls including those called by child programs.

#### `bpftrace` and `eBPF`

*`eBPF` (extended Berkeley Packet Filter) is a powerful Linux technology that allows running sandboxed programs in the kernel. `bpftrace` provides a high-level syntax for writing eBPF programs.* [![example of bpftrace][YT_ICON]](https://youtu.be/8VYT9TcUmKs?t=2203)

#### Network Debugging

Use `tcpdump` and `Wireshark`.

### Memory Debugging

#### Sanitizers

Sanitizers are like extensions to your compiler which make your compiler do sanity check to your program to prevent bad things.

An example of address sanitizer (`-fsanitizer=address`)—heap overflow. [![example of address sanitizer][YT_ICON]](https://youtu.be/8VYT9TcUmKs?t=2849)

#### Valgrind

`valgrind` is an extensible program interpreter. It pretends to be a CPU (so, it runs program slower than running at normal CPU). It takes and execute your program and it can check very memory manipulations that your program does.

It also gives people instruction-level information. For example, it can tell you how many cycles are taken to execute this function.

To check memory leak: `valgrind --leak-check=full ./my_program`. 

You can use valgrind even if you don't have source code.

### AI for Debugging

*AI can be an extremely useful tool for a certain subset of debugging.*

LLMs might not good at fixing bugs, but they are really good at finding them. And it's efficient to let LLMs figuring what's going on in codes that are unfamiliar to you.

## Profiling

Profiling is relevant to debugging.

### Timing

#### time

There is a tool called `time`, which will tell you how much time a command takes.

```bash
time curl https://missing.csail.mit.edu &> /dev/null

real    0m0.822s
user    0m0.025s
sys     0m0.027s
# Please note that sys time is CPU time, not real-world time
# For example, 10 CPUs runs the program for 1 ms, then sys time should be 10 ms
```

`time` is one of the easiest ways to profile two programs. But it's not reliable. Because the results will vary between executions.

#### hyperfine

You can give it multiple commands and it will run them multiple times and give you timing information.

For example:

```bash
# The following instruction compares the time consuming between
# [find . -name "*.md"]   and   [fd -e md]

hyperfine --warmup 3 'find . -name "*.md"' 'fd -e md'
Benchmark 1: find . -name "*.md"
  Time (mean ± σ):     731.7 ms ±  24.3 ms    [User: 176.0 ms, System: 482.4 ms]
  Range (min … max):   706.8 ms … 762.8 ms    10 runs

Benchmark 2: fd -e md
  Time (mean ± σ):      54.5 ms ±   2.9 ms    [User: 226.9 ms, System: 405.8 ms]
  Range (min … max):    48.8 ms …  61.3 ms    54 runs

Summary
  fd -e md ran
   13.43 ± 0.85 times faster than find . -name "*.md"

# Usually, fd is faster than find
```

### Resource Monitoring

`htop` is an improved version of `top`. `iotop` gives you information about input and output. `free` gives you information about memory usage. `lsof` gives you information about open files. `ss` gives you information about network connections. `nethogs` and `iftop` monitors each program uses how much network.

`btop` is an even better version of `htop`.

`perf` can tell you the which part of the program is spending the most time. Usage: `perf record -g ./my-program` (record what's going on in this program) + `perf record` (show the record).

`perf record` is hard to read and parse. `flamegraph` (`inferno-flamegraph` is the rust version of `flamegraph`) can help you with that. It will parse the `perf record` and generate a `.svg` graph to show which function takes most time. Usage: `perf script | inferno-collapse-perf | inferno-flamegraph > slow.svg` + `imv slow.svg`.

`perf` gives imperfect information because what it does is that it pauses the CPU frequently to see which part of the program is using the CPU currently. Although you can increase the sampling frequency to get more accurate results, the report cannot be 100% accurate. To get total details of your program, use Valgrind (it doesn't sample, it track each instruction).  But one thing is for sure, the more accurate result you need. the more time it takes.

### Visualizing Performance Data

To visualize how time increase with the scale of input, people invented some tools.

For example: `gnuplot`.

## In the End

Think about what you want to log and how you log early on.

## Tips

- `Ctrl + L` will clear the screen.

[YT_ICON]: https://img.shields.io/badge/YouTube-%23FF0000.svg?style=flat-square&logo=YouTube&logoColor=white

[MS_ICON]: https://missing.csail.mit.edu/static/assets/favicon-16x16.png

[MS_ICON_M]: https://missing.csail.mit.edu/static/assets/favicon-32x32.png
