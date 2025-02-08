# snow-crash

## About the project

**Snowcrash** is an introductory cybersecurity project from the **42Network** curriculum. It consists of **10 levels** and **5 bonus challenges**.

Each level is documented in its corresponding directory, with a detailed explanation of the approach and reasoning used to solve it.

## Table of contents

- [About the Project](https://www.notion.so/Leaffliction-11bd775f41e6809986ace63e12d25d2e?pvs=21)
- [Getting Started](https://www.notion.so/Leaffliction-11bd775f41e6809986ace63e12d25d2e?pvs=21)
- [Usage](https://www.notion.so/Leaffliction-11bd775f41e6809986ace63e12d25d2e?pvs=21)

## Getting Started

1. Create a virtual machine using the ISO file provided by 42. Ensure that the VM's network is set to **Host-Only Adapter**.
2. Find the IP address of your VM by running:

```
ifconfig
```

1. Connect via SSH on port 4242 replacing XX with the level you want to access and <VM_IP_ADDRESS> with your VM’s actual IP.

```python
ssh -p 4242 level00@<VM_IP_ADDRESS>
```

## Usage

1. **Log in to level00** using the default credentials:

```bash
ssh -p 4242 level00@<VM_IP_ADDRESS>
#password: level00
```

1. **Find the password for the next level.** 

Each level requires you to discover a password that grants access to the corresponding flagXX, where XX is the current level number. Once you've obtained the password for flagXX, log in as follows:

```
su flagXX
```

(You may not be able to connect to a "flagXX" account - in this case, you will have to find an alternative method).

1. **Retrieve the next password**

Once logged as flagXX, run the following command to obtain the token for the next level:

```bash
getflag
```

1. **Switch to the next level using the token as password**

```bash
su levelXX+1
```

### Ressources

**Tools**

- PCAP file reader and network protocol analyzer: wireshark
- Debugger: gdb
- Password cracking software tool: john

**Language:**

- python/ ruby script
- php/pearl/lua script

**Websites :**

- cloudshark.org
- dcode.fr
- duckduckgo.com
