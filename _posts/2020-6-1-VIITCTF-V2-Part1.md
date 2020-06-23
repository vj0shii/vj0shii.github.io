---
layout: post
title: WriteUp of VIIT CTF V2_Part-1
---
Hello, this is part 1 of the short writeup series of VIIT CTF V2

**Team** - N00bs | **Position** - 2nd (Runner ups) |
**Team Members:**

[vj0shii](https://vj0shii.github.io/about) - Vaibhav Joshi |
[p4nk4j](https://www.linkedin.com/in/p4nk4jv/) - Pankaj Verma |
[H4$H](https://www.linkedin.com/in/anuja-khandelwal-a83402182/) - Anuja Khandelwal |
[loopspell](https://www.linkedin.com/in/ankitkushwah/) - Ankit Kushwah

## Crypto

### Piggy

`piggy_32.txt`

```
$ file piggy_32.txt
piggy_32.txt: ASCII text
```

By name and data found it is base32 encoded

```
$ cat piggy_32.txt | base32 -d > piggy.txt
```

After this the data found was base64 encoded

```
$ cat piggy.txt | base64 -d > piggy
$ file piggy
piggy: PNG image data, 934 x 85, 8-bit/color RGB, non-interlaced
$ mv piggy piggy.png
```

after opening the png file with image viewer found another encoding, 

VIITCTF{--ENCODED--}

after some research found that, the encoding is pigpen

For decoding I used [this](https://crypto.interactive-maths.com/pigpen-cipher.html)

After decoding found the correct flag and submitted

VIITCTF{--PIGMENDECODED--}

### Twins

`twins.txt`

```
$ cat twins.txt
---..--....-.-...-.-....-...--.--...-..---..-...--..---.-..---...-.......--.-.-.....--....-..--..--..--....-.-
```

At first sight, it looks like morse code, but there are no spaces in this, which means that is we try to convert back there are thousands of possibilities

As the ctf name suggests a clue I started searching for the encodings similar to morse and came across Baudot which is similar to this, baudot also contains two characters 1 and 0 so I converted `- to 1` & `. to 0` and then decoded it with [this](https://www.dcode.fr/baudot-code)

and found the flag merged it with and submitted

VIITCTF{--BAUDOTDECODED--}

## Reverse

### App

`viitctf.apk`

Decompiling the apk with apktool

```
$ apktool d viitctf.apk
```

Searching for related strings in the code 

**I prefer grep**

```
$ cd viitctf
$ grep -rn "VIIT" .
./res/values/strings.xml:31:    <string name="app_name">VIIT CTF</string>
```

Found this strings inside strings.xml

strings.xml is the file where a developer can store all the necessary strings for the application and define a name for that string which can be used later in application

On looking at in the file the flag was saved in 5 different parts, combined the flag and submitted VIITCTF{flag1+flag2+flag3+flag4+flag5}

## Stego

### Alien

`alien.wav`

I used Stegnographic tool from [here](https://futureboy.us/stegano/decinput.html)

Found a string containig a number of 1 and 0, which was spoon binary language

Interpreted the code with spoon binary interpreter [here](https://www.dcode.fr/spoon-language)

Found the flag and submitted

### Shield

`shield.png`

Started [StegSolve](https://github.com/zardus/ctf-tools/tree/master/stegsolve) to analys the image

After opening the tool, changed the plains, and in `Blue Plain 0` Found the flag

## Binary

### rev1

`rev1_x64`

On executing the binary

```
$ ./rev
VIITCTF{flag}
```
Decompile the binary with Ghidra

found that there are 4 arrays, 1 which is being printed when running the binary `VIITCTF`

there were 3 more arrays with numbering 1, 2, 3 and assigned hex values to them

combining the hex string and decoding according to ASCII table, found the flag, combined with other part and submitted

VIITCTF{--FLAG--}

### rev2

`main.exe`

There was one more file inside the zip names func.pyc, so I used online decompiler to decompile that file and found the following code

```python
if len(sys.argv) < 2:
    print 'Give a string ', sys.argv[0], ' < string >'
elif len(sys.argv) < 3:
    print 'Give a number ', sys.argv[0], ' ', sys.argv[1], ' < number >'
else:
    flag(sys.argv[1], int(sys.argv[2]))
```

The flag function was

```
def flag(key, t):
    if key == 'Password' and t == 1337:
        part3 = 'flag part'
        print '404 Flag Not Found'
        part4 = 'flag part'
        time.sleep(60)
        flag = part1 + '_' + part2 + '_' + part3 + '_' + part4 + '_' + part5
        flag = flag.decode('rot13')
        print 'Just Kidding Flag is : VIITCTF{' + flag + '}'
    else:
        print key * t
```

So from this I understood that whenever flag function is called with first argument `Password` and second argument `1337` it will print `404 Flag Not Found`, and sleep for 60 seconds, after that it will concatinate all five parts of flag, then perform a rot13 decode, and print it in format, and as the main function programmed, we can call the flag function by passing more than 2 arguments

To exploit and get flag

```
$ main.exe Password 1337 exploit
404 Flag Not Found                //The code will stop for 60 seconds after this, soo....wait
VIITCTF{--FLAG--}
```
