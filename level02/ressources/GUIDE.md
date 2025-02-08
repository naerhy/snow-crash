Listing the files in the directory, we find a **.pcap** file:

```bash
level02@SnowCrash:~$ ls
level02.pcap
```

A **.pcap** file is a network traffic capture. It can be analyzed with **Wireshark**, which is not available on the **SnowCrash** VM.

To analyze the file, we transfer it to our local machine using **scp**:

```bash
level02@SnowCrash:~$ scp -P 4242 level02@localhost:/home/user/level02/level02.pcap ./
```

**Analyzing with wireshark**

Navigate to : analyze → follow → TCP stream

<aside>
💡

..wwwbugs login: l.le.ev.ve.el.lX.X
..
Password: ft_wandr...NDRel.L0L

</aside>

Wireshark provides multiple ways to interpret captured data, including **ASCII, UTF-8, and raw bytes**. Initially, the password extracted from **ASCII and UTF-8** appeared incorrect due to extra or corrupted characters:

**ASCII :** ft_wandr...NDRel.L0L

**UTF-8 :** ft_wandrNDRelL0L

To find the correct password, we manually translated the **raw byte values** to ASCII. The raw data was:

- **Raw**
    
    66 f
    74 t
    5f _
    77 w
    61 a
    6e n
    64 d
    72 r
    7f [DEL]
    7f [DEL]
    7f [DEL]
    4e N
    44 D
    52 R
    65 e
    6c l
    7f [DEL]
    4c L
    30 0
    4c L
    0d [end of transmission] 
    

The 7f [DEL] character is a delete/backspace control character, meaning some characters were removed when the password was typed.

By removing these **DEL** characters, we reconstruct the actual password:

**Password : ft_waNDReL0L**

```bash
level02@SnowCrash:~$ su flag02
password: **ft_waNDReL0L**
level02@SnowCrash:~$ getflag
Here is your token **kooda2puivaav1idi4f57q8iq**
```

### Ressources

- About .pcap file: https://fr.wikipedia.org/wiki/Pcap
- About Wireshark usage: https://www.varonis.com/blog/how-to-use-wireshark#what-is-wireshark
