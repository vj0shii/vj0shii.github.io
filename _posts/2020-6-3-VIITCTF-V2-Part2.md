---
layout: post
title: WriteUp of VIIT CTF V2_Part-2
---
Hello, this is part 2 of the short writeup series of VIIT CTF V2

**Team** - N00bs | **Position** - 2nd (Runner ups) |
**Team Members:**

[vj0shii](https://vj0shii.github.io/) - Vaibhav Joshi |
[p4nk4j](https://www.linkedin.com/in/p4nk4jv/) - Pankaj Verma |
[H4$H](https://www.linkedin.com/in/anuja-khandelwal-a83402182/) - Anuja Khandelwal |
[loopspell](https://www.linkedin.com/in/ankitkushwah/) - Ankit Kushwah
<!--more-->
**Note-** I havn't made notes for some of the challanges as I was in hurry to complete all, so there may be chances that some Strings or Filenames may be different from original, but the method is correct

## Web

### Crack It

For this challange, 1 file containing hash is provided, and a link to a login page

On researching found that the hashes are bcrypt encrypted, to decrypt I used [Decryptor](https://github.com/BREAKTEAM/Debcrypt)

```
python3 crack.py
You want crack? y/n y
hash to crack: --HASHHERE--
```

Decrypting both the hashes with this and logged in with found credentials, after login found the flag

### Fuzz It

For this challange a domain is provided, where we need to fuzz the directories, I used [dirsearch]() for this

after directory brute forcing found a number of directories, the hint is that, if you are in current directory you will see text `Fuzz Dipper` and if you are in incorrect directory, it will show something that `This is not the directory`

When you reach at last directory a text will show on the page like `The flag is in front of you`, open the source of the page and the flag is in html comment

### Shell It

A domain is provided to start the challange, on the domain homepage it is written that there is a cmd parameter which can be used to execute command, to execute `ls` visit

```
http://viitctf.digininjas.in:8003/?cmd=ls
```

there was a file names `flag.txt`, reading the file to get flag

```
http://viitctf.digininjas.in:8003/flag.txt
```

## Pwn

### FTP - Removed from the CTF

A domain is provided to start the challange, first started enumeration of ftp if any vulns exist or any misconfiguration but found nothing

On the webserver found robots.txt with a entry, on visiting that link, found base64 encoded credentials, decoded the strings and used as username and password to login into ftp

flag was in a file inside ftp, just get the file and read it

### SSh

After getting ftp access, found a ssh key on the server, with filename - `tonystark`, from that I guessed user as tonystark

when tried to login found that, there is a need of passphrase to login with the key

**cracking the passphrase**

I used two tools for this [ssh2john.py](https://fossies.org/linux/privat/john-1.9.0-jumbo-1.tar.xz/john-1.9.0-jumbo-1/run/ssh2john.py?m=b) & john, first convert hash from ssh key from ssh2john then crachek the hash with john
```
$ ssh2john.py tonystark > hash
$ john -w=rockyou.txt hash
```

Found passphrase with this and logged in with the key with below command

```
$ ssh -i tonystark tonystark@viitctf
passphrase:
```

On the home directory found the flag in a file

### SQL

After completing SSH there was a hidden file in the home directory `.db.php` which cotains code to connect with mysql server and containing bcrypt hashes of username & password, 

decrypted that as did in Crack It, and login in mysql server with the creds and get the flag

## Binary

### Rev3

Analysing the binary in radare2

```
$ r2 d ./rev3_x64
> aaa               analysing the binary
> afl               loading all module and listing
> s sym.print       Opening print function because it is the function looked suspecious to me after going through other functions
> pdf               To view the the assembly code
```

In the assembly code i found 3 strings , combined them in a single string & tried to submit it as flag but falied

After that for better understanding decompiled the binary with ghidra and viewed the source code manually, a small part of code is below, which is useful

```c
if (*(char *)(parm_1 + 1) == A) {
  printf("H_=70b9E0<cbC3");
  }
else {
  printf("13 ->")
}
```

After some hit and try got nothing, after some research for encoding I came across rot13 encoding, which is a hope as the else condition is "13 ->" which is similar, I reverted the strings, with the string in this code I found nothing, but remember the string we found in starting with radare2, when I reversed that, what I found looks like a sentence and made sense

So I combined it with VIITCTF to make standard format as from the code

VIITCTF{--ROT13DECODE--}

Submitted the flag successfully

### Rev4

**Working on it comming soon**
