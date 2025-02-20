Listing the files in the directory, we cannot find any binary file belonging to flag14. Inspired by the previous level, we try to modify the getflag function uid using gdb. 

```bash
**level14@SnowCrash:~$** gdb getflag
...
Reading symbols from /bin/getflag...(no debugging symbols found)...done.
(gdb) b getuid
Breakpoint 1 at 0x80484b0
(gdb) run
Starting program: /bin/getflag 
You should not reverse this
[Inferior 1 (process 3382) exited with code 01]
```

We attempted to set a breakpoint on the **getuid** function and modify the return value stored in the **$eax** register to the corresponding flag14 user ID. However,the program is protected and displays the message “You should not reverse this”. This is similar to the level09, where gdb usage was restricted by a ptrace call. 

Upon running gdb and setting a breakpoint at the ptrace system call, we confirm the protection by checking the value of **$eax**, which returns -1. This indicates that **ptrace()** is blocking debugging access.

The **ptrace()** system call is used by programs to detect if a debugger is attached. When gdb is used to debug a process, the process may call **ptrace()** with the **PTRACE_TRACEME** option, causing it to return -1 to indicate that tracing is not allowed.

In order to bypass this protection, we modify the value of **$eax,** which contains the return value of the last system call. Setting **$eax** to 0 effectively overrides the **ptrace()** return value, tricking the program into thinking no debugger is attached, and allowing gdb to continue manipulating the process.

```bash
**level14@SnowCrash:**/bin$ gdb getflag
...
Breakpoint 1, 0xb7f146d0 in ptrace () from /lib/i386-linux-gnu/libc.so.6
(gdb) stepi
0xb7f146d3 in ptrace () from /lib/i386-linux-gnu/libc.so.6
(gdb) step
Single stepping until exit from function ptrace,
which has no line number information.
0x0804898e in main ()
(gdb) p $eax
$1 = -1
(gdb) set $eax=0
(gdb) s
Single stepping until exit from function main,
which has no line number information.
```

Now that we have bypassed **ptrace()** and the debugger can continue to work, we set a breakpoint at the **getuid** function. and change the return value of the last system call to the UID of the flag14 user. This allows to execute with the appropriate permissions. 

By modifying the return value of **getuid** (stored in **$eax**), we can simulate the return of the flag14 user's UID, allowing us to execute the **getflag** command with flag14's permissions.

This step enables us to exploit the binary and retrieve the flag.

```bash
Breakpoint 2, 0xb7ee4cc0 in getuid () from /lib/i386-linux-gnu/libc.so.6
(gdb) stepi
0xb7ee4cc5 in getuid () from /lib/i386-linux-gnu/libc.so.6
(gdb) s
Single stepping until exit from function getuid,
which has no line number information.
0x08048b02 in main ()
(gdb) p $eax
$2 = 2014
(gdb) set $eax=3014
(gdb) c
Continuing.
Check flag.Here is your token : 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
[Inferior 1 (process 3356) exited normally]
```

Here is the full method:

```bash
level14@SnowCrash:/bin$ gdb getflag
...
Reading symbols from /bin/getflag...(no debugging symbols found)...done.
(gdb) break ptrace
Breakpoint 1 at 0x8048540
(gdb) break getuid
Breakpoint 2 at 0x80484b0
(gdb) run
Starting program: /bin/getflag 

Breakpoint 1, 0xb7f146d0 in ptrace () from /lib/i386-linux-gnu/libc.so.6
(gdb) stepi
0xb7f146d3 in ptrace () from /lib/i386-linux-gnu/libc.so.6
(gdb) step
Single stepping until exit from function ptrace,
which has no line number information.
0x0804898e in main ()
(gdb) p $eax
$1 = -1
(gdb) set $eax=0
(gdb) s
Single stepping until exit from function main,
which has no line number information.

Breakpoint 2, 0xb7ee4cc0 in getuid () from /lib/i386-linux-gnu/libc.so.6
(gdb) stepi
0xb7ee4cc5 in getuid () from /lib/i386-linux-gnu/libc.so.6
(gdb) s
Single stepping until exit from function getuid,
which has no line number information.
0x08048b02 in main ()
(gdb) p $eax
$2 = 2014
(gdb) set $eax=3014
(gdb) c
Continuing.
Check flag.Here is your token : 7QiHafiNa3HVozsaXkawuYrTstxbpABHD8CPnHJ
[Inferior 1 (process 3356) exited normally]
```

### Ressources

- About bypassing ptrace: [https://www.notion.so/snow-crash-17dd775f41e68097b62ac050b3473b11](https://www.notion.so/snow-crash-17dd775f41e68097b62ac050b3473b11?pvs=21)
