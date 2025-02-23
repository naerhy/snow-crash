# snow-crash

## About the project

**snow-crash** is an introductory cybersecurity project from the **42Network** curriculum. It consists of **10 levels** and **5 bonus challenges**.

Each level is documented in its corresponding directory, with a detailed explanation of the approach and reasoning used to solve it.

## Getting Started

1. Create a virtual machine using the ISO file provided by 42. Ensure that the VM's network is set to **Host-Only Adapter**
2. Find the IP address of your VM by running `ifconfig`
3. Connect via SSH on port **4242**, replacing XX with the level you want to access and <VM_IP_ADDRESS> with your VM’s actual IP

```bash
ssh -p 4242 levelXX@<VM_IP_ADDRESS>
```

## Usage

### Log in to levelXX using the known credentials:

```bash
ssh -p 4242 level00@<VM_IP_ADDRESS>
#password: level00
```
### Find the password for the next level

Each level requires you to discover a password that grants access to the corresponding flagXX, where XX is the current level number. Once you've obtained the password for flagXX, log in as follows:

```bash
su flagXX
```

You may not be able to connect to a "flagXX" account - in this case, you will have to find an alternative method.

### Retrieve the next password

Once logged as flagXX, run the following command to obtain the token for the next level:
```bash
getflag
```

### Switch to the next level using the token as password

```bash
su levelXX+1
```

### Ressources

#### Tools & Websites

- [dCode](https://www.dcode.fr)
- [GDB](https://en.wikipedia.org/wiki/GNU_Debugger)
- [John the Ripper](https://www.openwall.com/john)
- [Wireshark](https://www.wireshark.org)

#### Languages

- Bash
- C
- Lua
- Perl
- PHP
