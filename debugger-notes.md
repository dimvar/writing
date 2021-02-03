# Notes on debuggers

### How a debugger works in Linux

In Linux, a process can trace the execution of another process using the ptrace
system call. Tracing allows us to control the execution of the process: start
it, pause it, resume it, look at the values of registers, etc. Ptrace also
allows us to attach to an already executing process. When a process A traces a
process B, A is notified by the OS about every signal sent to B.

Debuggers implement their functionality (such as setting breakpoints) on top of
ptrace.

Breakpoints are implemented using software interrupts. When we set a breakpoint
at an instruction X of a program P, the one-byte x86 assembly instruction
`int 3` is inserted at that point in P. When P executes `int 3`, that is a
software interrupt that stops the execution of P, and the OS sends SIGTRAP to P.
Then, the debugger process gets notified about the signal (which lets it do some
bookkeeping). Then, the user can do the usual things, such as inspect the memory
of P at the breakpoint.

The debugger maintains a mapping between lines in the source program and the
corresponding assembly instructions. The mapping is needed because the user
works (e.g., sets breakpoints) in terms of source lines. To create the mapping,
the debugger uses information from the dwarf format. Dwarf is the debugging
format for linux. Dwarf information is embedded inside ELF files. Using dwarf,
we can find where each function starts in the assembly, where each variable is
stored in memory (so the user can inspect its value), etc. If you want to peruse
the dwarf info yourself, you use `objdump`, e.g.:  
`$ objdump --dwarf=info my_executable_name`

### strace basics

strace can attach to a process and show you the system calls performed by that
process.
You can either start a process with strace  
`$ strace my_command`  
Or you can strace a running process  
`$ strace -p PID`

### gdb basics

To see the stack trace of a running process use bt (stands for backtrace)
```bash
gdb
(gdb) attach PID
(gdb) bt
```

### References
Eli Bendersky's series of posts on debugging:
[part1](https://eli.thegreenplace.net/2011/01/23/how-debuggers-work-part-1/),
[part2](https://eli.thegreenplace.net/2011/01/27/how-debuggers-work-part-2-breakpoints/),
[part3](https://eli.thegreenplace.net/2011/02/07/how-debuggers-work-part-3-debugging-information).

