---
layout: post
title: Testing WebApp Access Control with Autorize
---
This blog is about [Autorize](https://github.com/Quitten/Autorize) which is an Automated Authorization testing plugin for burp suite, the tool can be used to check if proper Access control is implemented on the server or not
<!--more-->
## What is Authorization/Access Control

Authorization or Access control are the rules or privileges which are set for every user role, the rules define that if a particular user or a member of a group have the permission to access any functionality on the server or not, if the permission to alloted then server will response back with the content but if the user does not have privileges then the server reply that the access is forbidden and mostly with 403 response code

## What is Access Control Bypass/Privilege Escalation

This is a condition where the Access rules are not configured or improperly configure on the server side which eventually allows the user to access the functionality which, it should not

## Why Autorize

As a Security researcher, when you test a application, many times the application scope is very wide and having hundreds and thousands of pages in it, in this case it is very difficult to test the rules for each page and eventually many pages are missed out, whereas Autorize automatically test the access control while you are browsing the application

## How to use Autorize

The installation steps can be found on the github repository of the tool, after successfully installing the plugin in burp suite, open the Autorize tab

First login with the lower privilege user from your browser and intercept the request and copy the token or cookie and place it in the header box inside Configuration tab of Autorize or just intercept a request and click on the button "Fetch cookie from last request" from Configuration tab

Then turn off intercept for that user and in new browser login with higher privilege user and start intercepting the requests of privilged functionality like admin-panel

**Note** - Don't logout from another account as the cookie in Autorize will Expire

Now browse the site with admin user and after that look at the Autorize tab if any of the functionality can be used by a lower privilege user it will show *Bypassed* for that request in Autorize, to view complete request open the Request/Response Viewer tab

![requests in Autorize](/images/blogs/Autorize.png)

**TIP** : Authentication can also be tested at some extend from this by selecting the *Check Authenticated" checkbox in the Configuration tab which will send the request without any session cookie or token
