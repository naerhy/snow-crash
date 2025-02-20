# Level13

## Walkthrough

We list the files in the home directory.

```bash
level13@SnowCrash:~$ ls -la
total 20
dr-x------ 1 level13 level13  120 Mar  5  2016 .
d--x--x--x 1 root    users    340 Aug 30  2015 ..
-r-x------ 1 level13 level13  220 Apr  3  2012 .bash_logout
-r-x------ 1 level13 level13 3518 Aug 30  2015 .bashrc
-r-x------ 1 level13 level13  675 Apr  3  2012 .profile
-rwsr-sr-x 1 flag13  level13 7303 Aug 30  2015 level13
level13@SnowCrash:~$ file level13 
level13: setuid setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0xde91cfbf70ca6632d7e4122f8210985dea778605, not stripped
```

We run the file.

```bash
level13@SnowCrash:~$ ./level13 
UID 2013 started us but we we expect 4242
```

We analyze the code of the `level13` executable.

[Link to the decompiled executable](https://dogbolt.org/?id=71a282f9-4824-48f8-a43a-b5f99d97184a).

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __uid_t v3; // eax
  char *v4; // eax

  if ( getuid() != 4242 )
  {
    v3 = getuid();
    printf("UID %d started us but we we expect %d\n", v3, 4242);
    exit(1);
  }
  v4 = ft_des("boe]!ai0FB@.:|L6l@A?>qJ}I");
  return printf("your token is %s\n", v4);
}
```

```c
char *__cdecl ft_des(char *s)
{
  unsigned int i; // [esp+2Ch] [ebp-1Ch]
  int v3; // [esp+30h] [ebp-18h]
  int j; // [esp+34h] [ebp-14h]
  int k; // [esp+38h] [ebp-10h]
  char *v6; // [esp+3Ch] [ebp-Ch]

  v6 = strdup(s);
  v3 = 0;
  for ( i = 0; strlen(v6) > i; ++i )
  {
    if ( v3 == 6 )
      v3 = 0;
    if ( (i & 1) != 0 )
    {
      for ( j = 0; *(char *)(v3 + 134514368) > j; ++j )
      {
        if ( ++v6[i] == 127 )
          v6[i] = 32;
      }
    }
    else
    {
      for ( k = 0; *(char *)(v3 + 134514368) > k; ++k )
      {
        if ( --v6[i] == 31 )
          v6[i] = 126;
      }
    }
    ++v3;
  }
  return v6;
}
```

The executable:
- checks if the current user id is equal to `4242`
- calls a custom `ft_des()` function which seems to decrypt the value `boe]!ai0FB@.:|L6l@A?>qJ}I`

Our first attempt in order to get the flag is to copy/paste the `ft_des()` function, and run it on our system with the same value.

```bash
./a.out 
Segmentation fault (core dumped)
```

The `level13` executable uses the number `134514368` to dereference a pointer. There might be a reason this is only working on the VM.

The only remaining instruction which looks exploitable is the `getuid()` call. After some researches, we find that we can alter the return value of a function with GDB.  
We try to replace the return value from `getuid()` with `4242`.

```bash
level13@SnowCrash:~$ gdb level13 
GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "i686-linux-gnu".
For bug reporting instructions, please see:
<http://bugs.launchpad.net/gdb-linaro/>...
Reading symbols from /home/user/level13/level13...(no debugging symbols found)...done.
(gdb) break getuid
Breakpoint 1 at 0x8048380
(gdb) r
Starting program: /home/user/level13/level13 

Breakpoint 1, 0xb7ee4cc0 in getuid () from /lib/i386-linux-gnu/libc.so.6
(gdb) s
Single stepping until exit from function getuid,
which has no line number information.
0x0804859a in main ()
(gdb) p $eax
$1 = 2013
(gdb) set $eax=4242
(gdb) c
Continuing.
your token is 2A31L79asukciNyi8uppkEuSx
[Inferior 1 (process 3834) exited with code 050]
```

## Resources

- [Modify-function-return-value Hack! — Part 1](https://www.opensourceforu.com/2011/08/modify-function-return-value-hack-part-1/)
- [GDB Quick Reference](https://users.ece.utexas.edu/~adnan/gdb-refcard.pdf)
