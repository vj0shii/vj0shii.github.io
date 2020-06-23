---
layout: post
title: Reviewing the Frontend Source Code of Web Application
---

After my last blog [Chaining Small Vulnerabilities to Account takeover](https://vj0shii.github.io/chaining-vulnerabilities-to-compromise-account/), I got many messages asking, how I found the variable from javascript, so, in this blog I will demonstrate, how do I review the frontend source code, this does not include analysis from any tools<!--more-->, but some good tools are DumpsterDiver, truffleHog, etc.

Nowadays everyone needs a website, which is user friendly and process every function as quickly as possible, also JavaScript and HTML5 offers a wide range of functions that can be performed easily at client-side nowadays, so developers uses the frontend code to do many operations, but the problem occurs when the developer stores sensitive data in frontend code, like API keys, credentials, etc., or perform sensitive functions only on client-side

### Debugging with Browser Developer tools

Open the Browser Developer tools by pressing `ctrl+Shift+I` in browser

**Searching for sensitive information**

Open the console tab and

```
> console.log(
```

The console will start suggesting you the varibales avaiable in javascript, start searching for variables similar to sensitive information like, txt, text, pass, user, api, key, creds

Or in the console start typing names of functions directly,

```
> FUNCTION_HERE()
```

which looks like sensitive functions like, reset, rst, change, chg, password, pass, creds, update, api

### Static Code Analysis

save complete frontend source code with wget recursive download or any other tool, I prefer wget

start searching for all .js included with any text editor, I use grep & Notepad++, we can search with regex queries or with strings inside a bunch of file/folder, Notepad++ has a very good `Find in files` option from where you can search a string or regex & strings
```regex
https.*?.js
```

Searching with this regex we can find almost all the js code, externally included in the application, save the URLs in a file and use wget to download all

```
wget -i file.txt
```

Or you can get all js URLs with burpsuite, for that start burp proxy and set the browser to send all requests through the proxy, after walking through all the functions, Go to `Proxy > HTTP history` and in filter select `Filter by file extension` and in the block enter `js`

After this burp will list all js files, select all Right-click and select `Copy URLs`, paste it in a file and use wget to download all files

**Unminify or Deobfuscate Scripts**

I use online platform [Unminify](https://unminify.com/) for this, just open the link copy paste the scripts and click on unminify to get original js code, which can be reviewed

**Code Review**

Then copy all files contains, found working code in a single directory, and use Notepad++ to search for sensitive strings or dangerous functions, this includes data in HTML code which we can miss with developer tools

Strings

* txt, text, pass, user, api, key, creds, etc.

Functions

* eval, anyElement.innerHTML, etc.

This was a basic review of frontend source code, I haven't mentioned any tools as I think the tools descriptions are enough for anyone to use
