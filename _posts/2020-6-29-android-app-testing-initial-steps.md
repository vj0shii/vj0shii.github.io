---
layout: post
title: Android App Source code Extraction and Bypassing Root and SSL Pinning checks
---

In this blog I will be describing the pre-requesty steps I followed for one of the android application penetration testing<!--more--> which includes
 
 * [Source Code Extraction](https://github.com/vj0shii/vj0shii.github.io/blob/master/_posts/2020-6-29-android-app-testing-initial-steps.md#source-code-extraction)
 
 * [Bypass Root Detection & SSL Pinning](https://github.com/vj0shii/vj0shii.github.io/blob/master/_posts/2020-6-29-android-app-testing-initial-steps.md#bypass-root-detection--ssl-pinning)
 
 Below are the steps with description
 
 <h2>Source Code Extraction</h2>
 
 <h3>Extracting jar file<h3>
 
 As we know that the apk is also a archive file so we can simply rename it to a zip and extract the data
 
 ```
 $ mv test.apk test.zip
 $ unzip test.zip
 ```
 
 After this there will be a classes.dex files, there can be more than one, like, classes2.dex, the dex files can be converted to jar file with `dex2jar`, you can install it in debian linux machine with the command below
 
 ```bash
 $ apt install dex2jar
 ```
 
 For converting a dex file into a jar, use below command
 
 ```
 $ d2j-dex2jar classes.dex
 ```
 
 after this a new jar file will be created in current directory
 
 ### Deobfuscating code in jar file if obfuscated
 
**Obfuscation** is a process which changes the functions and varibale names and created a lot of classes to jump around in the code, which makes it difficult to understand. Programmers may deliberately obfuscate code to conceal its purpose (security through obscurity) or its logic or implicit values embedded in it, primarily, in order to prevent tampering, deter reverse engineering, or even as a puzzle or recreational challenge for someone reading the source code.

To deobfuscate code I mainly use tool called [deobfuscator](https://github.com/java-deobfuscator/deobfuscator/releases/download/1.0/deobfuscator.jar)

To deobfuscate code download the tool from above link and follow below steps

**Identifying obfuscator**

Create a yaml file with below content for now let the file name `test.yml`

```yaml
input: classes.jar
detect: true
```

This will a output like below

```
[Thread-0] INFO com.javadeobfuscator.deobfuscator.Deobfuscator - Loading classpath
[Thread-0] INFO com.javadeobfuscator.deobfuscator.Deobfuscator - Loading input
[Thread-0] INFO com.javadeobfuscator.deobfuscator.Deobfuscator - Detecting known obfuscators
[Thread-0] INFO com.javadeobfuscator.deobfuscator.Deobfuscator - 
[Thread-0] INFO com.javadeobfuscator.deobfuscator.Deobfuscator - WhateverFunc: Some obfuscators don't remove the SourceFile attribute by default. This information can be recovered, and is very useful
[Thread-0] INFO com.javadeobfuscator.deobfuscator.Deobfuscator - 	Found possible SourceFile attribute on ----
[Thread-0] INFO com.javadeobfuscator.deobfuscator.Deobfuscator - Recommend transformers:
[Thread-0] INFO com.javadeobfuscator.deobfuscator.Deobfuscator - 	com.javadeobfuscator.deobfuscator.transformers.normalizer.SourceFileClassNormalizer
```

There can be more than one transformer take a note of this

**Deobfuscating jar**
Create a new yaml file config.yml, the name needs to be same for this

```yaml
input: classes.jar
output: classes-deobfuscated.jar
transformers:
  -com.javadeobfuscator.deobfuscator.transformers.normalizer.SourceFileClassNormalizer
  -anotherTranformer
```

The above listed transformer during obfuscator detection phase, there can be more than one transformers which can be listed each on a new file like the one lister above

Sometimes there are multiple layer of obfuscation, in that case above steps can be followed recursivly to get final jar with readable code

Finally open jd-gui and open the final jar, on the top-left corner inside file, click on `Save all Sources`, to save java code a zip file will be saved containing all the java code

## Bypass Root Detection & SSL Pinning

### Root Detection
So there was root detection and ssl pinning in the application which needs to be bypassed for further testing, I started searching for function names similar to root or detection, and found a function named `RootDetection()` let be in a file StartActivity.java on the top level directory, because the file I found is named respetive to the company app name

So functions structure was like, it is performing several checks and returning the final response as true or false, true means device is rooted, false means it is not

```java
RootDetection() {
  if (new File("/usr/bin/su").exists()) continue:
      return true;
  --More if statements--
  return false
```

To bypass above protection I used Frida

Download frida server from [here](https://github.com/frida/frida/releases/)

To get the architecture of device, connect device to the host system and run below command

```
adb shell getprop ro.product.cpu.abi
```

unpack the xz file and transfer the server file to device, and start server with below commands

```
$ adb push frida-server /data/local/tmp/
$ adb shell
> cd /data/local/tmp
> chmod +x frida-server
> ./frida-server
```

Then i created a frida script to bypass root detection according to the application, which is a JavaScript file, names test.js

```javascript
setTimeout(function(){
	Java.perform(function (){
		console.log("[*] Script loaded")

		var MenuActivity = Java.use("sg.vantagepoint.mstgkotlin.MenuActivity")

		StartActivity.RootDetection.overload().implementation = function() {
			console.log("[*] RootDetection function invoked")
			return false
		}
	});
});
```

For running the script install frida in your host machine, which can be installed from python-pip

```
$ pip install frida-server
```

Basic Usage

```
Connect frida to USB device
$ frida-ps -U
List running applications
$ frida-ps -Ua
List installed applications
$ frida-ps -Uai
```

Install the application in the device

Finally running the created script script

```
$ frida -U -l test.js -f *app_name* --no-pause 
```

You can get the app name by listing all installed applications, After running above steps, it will automatically restart the application and the root detection will be passed, applicaton can be successfully used in the rooted device

### SSL Pinning

So after bypassing the root detection, the problem was the ssl pinning I used the [Univeral SSL Pinning bypass script](https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/)

Intially as I will be using burp suite for testing, I used burpsuite CA cert for this, `cacert.der`

Transfer this cert to the device, the location is specified is script, if you modify the file location or name remember to specify in script(line 30)

```
$ adb push cacert.der /data/local/tmp/cert-der.crt
```

Start the frida server in device and run below command

```
$ frida -U -l ssl-bypass.js -f *app_name* --no-pause 
```

The `ssl-bypass.js` is the file we downloaded previously

With this the SSL Pinning can be successfully bypassed, but as there is root detection so we need to combine both scripts into single to bypass both the checks, the final script which worked for me is [here](https://gist.github.com/vj0shii/1572f568e883e2fbfcb172e4f4bdf892)



**Note:** Many times there will be problems with the jar extracted from dex2jar in that can you can use [simplify](https://github.com/CalebFenton/simplify) or [dex-oracle](https://github.com/CalebFenton/dex-oracle)
