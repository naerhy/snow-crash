# Snow Crash

> This project is an introduction to computer security. Snow Crash will help you discover security in various sub-domains, with a developer-oriented approach. You will become familiar with several languages (ASM, Perl, Php, etc.), develop a certain logic to understand unknown programs, and become aware of problems related to simple programming errors.

## Description

Snow Crash is the first project of the cybersecurity branch of 42 school. It serves as an introduction to some well-known system vulnerabilities, and mostly focuses on the Linux operating system, forcing us to learn more about it and understand how it can be potentially exploited.

The main topics we have studied include:
- Linux filesystem and commands
- Users permissions
- Shell and Perl scripting
- C and PHP executables

All the levels and bonuses of this project have been completed.

## Usage

In order to start this project, you have to first download and install the provided ISO by 42.  
Then you need to use a virtual machine with the ISO. If using **VirtualBox**, you must set the `Attached to` setting in the `Network` tab to `Host-only Adapter`.  
Finally you can connect to the first level, using ssh. The ip is displayed on the home screen of the VM and the port is `4242`. The credentials for the first level are: `level00` for both username and password.

```bash
ssh level00@xxx.xxx.xxx.xxx -p 4242
```

Once you have found the password for the next level, switch to the corresponding `flag` user and run the `getflag` command in order to obtain the password required to log in as the next `level` user.

```
level00@SnowCrash:~$ su flag00
Password:
Don't forget to launch getflag!
flag00@SnowCrash:~$ getflag
Check flag.Here is your token: ?????????????????
flag00@SnowCrash:~$ su level01
Password:
level01@SnowCrash:~$ _
```

Logging in as a `flag` user is not always required, sometimes you may find another solution in order to directly obtain the next `level` password.
