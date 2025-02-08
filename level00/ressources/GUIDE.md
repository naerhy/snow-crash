To find the first flag, we use the find command to search for a file owned by flag00 that we have permission to read:

```bash
level00@SnowCrash:~$ find / -type f -user flag00 2>dev/null
```

This reveals a file named john located at **/usr/sbin/john**.

```bash
level00@SnowCrash:~$ cat /usr/sbin/john
cdiiddwgpswtgt
```

Using this string as the flag does not work. It is encoded using a **Caesar cipher**. We decode it using decode.org. 

**Password: nottoohardhere**

```bash
level00@SnowCrash:~$ su flag00
password: nottohardhere
level00@SnowCrash:~$ getflag
Here is your token x24ti5gi3x0ol2eh4esiuxias
```

### Ressources

**Caesar code** : [https://fr.wikipedia.org/wiki/Chiffrement_par_décalage](https://fr.wikipedia.org/wiki/Chiffrement_par_d%C3%A9calage)

**Tool to decrypt ceasar code** : dcode.fr
