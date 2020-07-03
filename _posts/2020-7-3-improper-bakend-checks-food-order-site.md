---
layout: post
title: Price Tampering due to Improper checks on applying Coupon
---
I was recently hunting on a target, which is a food ordering site and found a bug from which I can tamper price during food order<!--more-->

There was one option to generate coupon by which I can generate coupon of whatever the amount I want and  it will provide me a 16 digit number which I can use to add the coupon during payment to reduce price at that time, similar case like a Gift card

In starting when i was looking I found out no rate limit on that gift card validation request, and with that i found so many gift cards, but that was a normal finding and they implemented CAPTCHA the next day, I was looking for some interesting findings and found a recent blog about tampering price with coupon code due to improper checks

I read that blog and tried a similar approach, I generated a coupon and noted down the number, after that I tried to apply that during a order, there were two requests

**Coupon apply**

```
POST /order/validate/couponCode HTTP/1.1
HEADERS_HERE
COOKIES

{"couponNumber":"1234567812345678"}
```

This request validates the coupon number and if the card is valid, applies discount, in response I got a Hash, which was then sent with the payment request

**Payment Code**

```
POST /order/payment HTTP/1.1
HEADERS_HERE
COOKIES

{"ordID":"...","email":"...","coupon":"HASH-HERE"}
```

After a successful payment request and coupon code expires, so below is the code flow according to me

Generate Coupon -> Apply Coupon -> Validate Coupon -> Apply Discount -> Payment -> Expire Coupon -> Expire Hash

So from this knowledge, I opened two tabs and added some items proceed to payment and added same coupon on both the sides, the thing I noticed is, I got two different Hashes for both requests

Then I completed payment in one tab and after that in another tab, both payments processed successfully with the discount

### One more interesting finding in the same

As I explained the code flow the Coupon code expires and after that the hash expires, so I thought that may be there is a chance that as the coupon number for second request is already expired, the code to expire that will fail, and so may be the code after that

So the Hash I had for second request, I used it again on another payment and it worked, the hash was not expired even after many requests

As I was curious I asked the company about it and they provided some information about it, a sample code is below

```php
ProcessHash() {
	//Extract Coupon Number with HASH
	if (!(Valid($CNumber))) {
		exit("Invalid Coupon");
	}
	ExpireHash();
}
```

As the if condition will fail so the ProcessHash fuction will not be processed futher and the code after that will be skipped, due to which the Hash will not be Expired

### Impact

Create a $50 coupon and apply it in two payments, Money spend = $50, Discount = $100, the compant is in loss of $50