---
layout: post
title: Multi-factor Auth Bypass with Password Reset Function
---

Recently when I was testing a web application, which is used for money transfer, wallet and banking, the scenario was that the multi factor authentication was compulsory, without enabling the user cannot use the application, I was able to login into someone's account without MFA and with only username and password
<!--more-->

So I had one account which was setup with MFA, during enumeration when i was testing the password reset function, i found that during password reset request a token in base64 format is generated and sent to reset the password the link was similar to

```
https://test.com/resetPass/{--BASE64Token--}
```

When I decoded the base64 token i found `Pentest-HFJDIAJD849F1575CCFBF3A4D6CF00E6C5641B7FD4DA2ED3E212C2D79BA9161A5A432FF0UFJDHFJD`, in this `Pentest` was my username but I was not able to understand the random string after, and thought it is just some random thing, i tried to create several tokens and came to know that it is always different, initally I ignored it and tried testing other thing, but as I normally do i had saved all the tokens and information in my note making application, tried several other things and slept after that

The next day when I was going through my notes for something and saw something similar in all the tokens, only the starting and ending 8 digits were random rest the string was same for all the links, I removed the random string from token and searched for it `849F1575CCFBF3A4D6CF00E6C5641B7FD4DA2ED3E212C2D79BA9161A5A432FF0`, eventually I found out that it is SHA-256

**My Approach towards HASH**

As I know it will take so much time to decrypt it and also not even possible in many cases, when i work on any program i save all my input and all user identifiers in a file, like username, password, userId, etc., so I quickly created a python script which takes a file and string as input, generate SHA-256 for each word and match that with he token given in argument, I was not so sure as in many cases it don't work but in this case it did and i found out the it is SHA-256 of my password which is `Test@1234`

I was not happy with this because it is just a Low finding and doesn't worth

**Taking a Turn**

After all this, next day, I started digging app again, and when I was logging in, the application asked for `Authenticator Token` for MFA. And it trigerred my mind

i sent a request to reset my password and accessed the link sent on the mail, opened it changed the password, and after changing it the app does not ask to login and user will be login directly after changing password. If i can generate the token with username and password I can bypass MFA, I generated the SHA-256 and concatinated it with the 8 random characters at start and end of it and then generated the base64 token as of my knowledge and tried to visit

```
https://test.com/resetPass/{--MyToken--}
```

When i visited I got response `no such token exist`. I started thinking again and after that first visited the password reset page, sent a request for password reset, then visited above link again and boom!!!!!! It worked I was able to change password and login in app

Interesting Fact: There was no old password check that means I can change the password again to `Test@1234` so user will not be able to find out
