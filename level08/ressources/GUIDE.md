Listing the files in the directory, we find a binary file:

```bash
level06@SnowCrash:~$ ls -la 
-rwsr-s---+ 1 flag08  level08 8617 Mar  5  2016 level08
-rw-------  1 flag08  flag08    26 Mar  5  2016 token
```

The s in **level08** permissions indicates the SetUID bit. This means the file will run with the privileges of its owner instead of the user executing it. In our case, the owner is **flag08.**

To analyze the binary file, we transfer it to our local machine using **scp**:

```bash
scp -P 4242 level08@127.0.0.1:/home/user/level08/level08 ./
```

We decompile the binary using [dogbolt.org](http://dogbolt.org) : 

https://dogbolt.org/?id=dd39ba2e-06a7-4e59-b781-7ddd00932474#Ghidra=198&Hex-Rays=133

![image.png](attachment:0852711d-bebd-487a-9d8f-649e64632dfe:image.png)

This script seems to open a file and read it, except if the file is named token, which is our case. 

If the script checks the filename before reading the file, then we could maybe use a symbolic link to bypass this condition. A **symbolic link** in Linux is a special file that points to another file or directory, like a shortcut.
