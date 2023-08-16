# To start off

You will need to connect to bandit servers via ssh banditX@bandit.labs.overthewire.org -p 2220, where X is the level that you want to connect to. After you've found the password for each level you can type 'exit' to start over and go to the next level.

To get a description of the command you can type "man" before the commando and a summary will show up. Ex:

```bash
man ls
```
You can also type 'reset' to clear the terminal for easier reading

```bash
reset
```

# Bandit level 0

The goal of this level is for you to log into the game using SSH. The host to which you need to connect is bandit.labs.overthewire.org, on port 2220. The username is bandit0 and the password is bandit0. Once logged in, go to the Level 1 page to find out how to beat Level 1.

### Commands you may need to solve this level
ssh - SSH (Secure Shell) is a protocol that allows secure remote access and command execution on a remote server over an encrypted connection.

---
```bash
cat readme
```

# Bandit level 1

The password for the next level is stored in a file called - located in the home directory

### Commands you may need to solve this level

ls , cd , cat , file , du , find

---
```bash
cat < -
```

# Bandit level 2

The password for the next level is stored in a file called spaces in this filename located in the home directory

### Commands you may need to solve this level

ls , cd , cat , file , du , find

---
```bash
cat "spaces in this filename"
```

# Bandit level 3

The password for the next level is stored in a hidden file in the inhere directory. "-a" shows hidden files, then you can just read them using cat

### Commands you may need to solve this level

ls , cd , cat , file , du , find

---
```bash
ls -a
cat .hidden
```

# Bandit level 4

The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.

### Commands you may need to solve this level

ls , cd , cat , file , du , find

---
```bash

```





# Bandit level 



### Commands you may need to solve this level



---
```bash

```
