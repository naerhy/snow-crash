# Level 09

## Waklthrough

For the last mandatory level, we keep listing the files in the current home directory first.

```bash
level09@SnowCrash:~$ ls -la
total 24
dr-x------ 1 level09 level09  140 Mar  5  2016 .
d--x--x--x 1 root    users    340 Aug 30  2015 ..
-r-x------ 1 level09 level09  220 Apr  3  2012 .bash_logout
-r-x------ 1 level09 level09 3518 Aug 30  2015 .bashrc
-r-x------ 1 level09 level09  675 Apr  3  2012 .profile
-rwsr-sr-x 1 flag09  level09 7640 Mar  5  2016 level09
----r--r-- 1 flag09  level09   26 Mar  5  2016 token
level09@SnowCrash:~$ file level09 
level09: setuid setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0x0e1c5a0dfb537112250e1c78d5afec3104abb143, not stripped
level09@SnowCrash:~$ file token 
token: data
```

There are 2 files: an executable `level09` with the **setuid** bit flag, and a `token` text file, both owned by **flag09** but readable.

```bash
level09@SnowCrash:~$ ./level09 
You need to provied only one arg.
level09@SnowCrash:~$ ./level09 token
tpmhr
level09@SnowCrash:~$ cat token 
f4kmm6p|=�p�n��DB�Du{��
```

From our first tests, it looks like the executable is accepting 1 argument and generate a string from it, perhaps a token.  
And we have to understand why the content of `token` contains some unprintable characters.

Let's import the executable with **SCP** and analyze it with **dogbolt**.

```bash
host:~$ scp -P 4242 level09@localhost:level09 level09
```

[Link to the decompiled output](https://dogbolt.org/?id=173c6b90-e0d8-4886-9fde-c92330912500)

We check the `main()` function and figure out that the program:
- prevents the executable to be called with a debugger like **GDB**, or to override a function with `LD_PRELOAD` environment variable
- accepts only 1 argument
- loops over the string passed as argument and prints on stdout the current character incremented by the current value of the loop counter variable
- prints a final newline character on stdout before exiting

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int result; // eax
  int v4; // [esp+20h] [ebp-118h]
  int v5; // [esp+24h] [ebp-114h]
  int v6; // [esp+28h] [ebp-110h]
  int v7[67]; // [esp+2Ch] [ebp-10Ch] BYREF

  v7[64] = __readgsdword(0x14u);
  v5 = 0;
  v4 = -1;
  if ( ptrace(PTRACE_TRACEME, 0, 1, 0) >= 0 )
  {
    if ( getenv("LD_PRELOAD") || open("/etc/ld.so.preload", 0) > 0 )
    {
      fwrite("Injection Linked lib detected exit..\n", 1u, 0x25u, stderr);
      return 1;
    }
    else
    {
      v6 = syscall_open("/proc/self/maps", 0);
      if ( v6 == -1 )
      {
        fwrite("/proc/self/maps is unaccessible, probably a LD_PRELOAD attempt exit..\n", 1u, 0x46u, stderr);
        return 1;
      }
      else
      {
        while ( 1 )
        {
          result = syscall_gets((int)v7, 256, v6);
          if ( !result )
            break;
          if ( isLib(v7, (int)"libc") )
          {
            v5 = 1;
          }
          else if ( v5 )
          {
            if ( isLib(v7, (int)"ld") )
            {
              if ( argc != 2 )
                return fwrite("You need to provied only one arg.\n", 1u, 0x22u, stderr);
              while ( ++v4 < strlen(argv[1]) )
                putchar(v4 + argv[1][v4]);
              return fputc(10, stdout);
            }
            if ( !afterSubstr(v7, (int)"00000000 00:00 0") )
              return fwrite("LD_PRELOAD detected through memory maps exit ..\n", 1u, 0x30u, stderr);
          }
        }
      }
    }
  }
  else
  {
    puts("You should not reverse this");
    return 1;
  }
  return result;
}
```

Out of curiosity, we pass the content of `token` to the executable.

```bash
level09@SnowCrash:~$ ./level09 `cat token`
f5mpq;v�E��{�{��TS�W�����
```

This doesn't seem to be the solution, and it can't be that simple.  
What's more likely is that the content of `token` is the result of the **flag09**'s password passed to the executable.  
The solution: create a small program to revert the token.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char** argv) {
  if (argc == 2) {
    size_t len = strlen(argv[1]);
    char* output = malloc(sizeof(char) * (len + 1));
    if (!output) return 1;
    output[len] = '\0';
    for (size_t i = 0; i < strlen(argv[1]); i++) {
      output[i] = argv[1][i] - i;
    }
    printf("output: %s\n", output);
  }
  return 0;
}
```

```bash
level09@SnowCrash:~$ mkdir /tmp/level09 && cd /tmp/level09
level09@SnowCrash:/tmp/level09$ vim script.c
level09@SnowCrash:/tmp/level09$ gcc -std=c99 script.c 
level09@SnowCrash:/tmp/level09$ ./a.out `cat ~/token`
output: f3iji1ju5yuevaus41q1afiuq
level09@SnowCrash:/tmp/level09$ su flag09
Password: 
Don't forget to launch getflag !
flag09@SnowCrash:~$ getflag
Check flag.Here is your token : s5cAJpM8ev6XHw998pRWG728z
```
