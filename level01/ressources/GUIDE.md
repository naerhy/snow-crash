```bash
level01@SnowCrash:~$ cat /etc/passwd
# 42hDRfypTqqnw
```
Using this string as the flag does not work. 

To analyze the file, we transfer it to our local machine using **scp**:

```bash
level01@SnowCrash:~$ scp -P 4242 level02@localhost: !!!!! remplacer par le bon path ./
```

Using john the ripper on /etc/passw

```bash
john /etc/passwd
```

su flag01 → abdcedf
