---
layout: post
title: "API Testing"
date: 2026-07-29

category: PortSwigger

image: /assets/posts/access-control-vulnerabilities/cover.png

excerpt: "PortSwigger Web Security Academy API Testing labs write-up with practical notes and exploitation steps."

tags:
- PortSwigger
- Web Security
- API
---

## Introduction

API Testing is the process of evaluating an application's Application Programming Interface (API) to identify security vulnerabilities and verify its functionality. It helps ensure that API endpoints handle requests securely, enforce proper authentication and authorization, and protect sensitive data from unauthorized access.

## Labs


### 01Exploiting an API endpoint using documentation

I captured the email update request in Burp Suite. I checked the API endpoint and the response. This helped me understand how the API works.

<img src="/assets/posts/apitesting/image.png" width="700" >

I opened the API documentation. I checked the available endpoints and HTTP methods. I found the DELETE endpoint for deleting a user.

<img src="/assets/posts/apitesting/image (1).png" width="700" >

I sent the **`DELETE /api/user/carlos**` request. The server deleted the user successfully. The lab was solved.

<img src="/assets/posts/apitesting/image (2).png" width="700" >


### 02. Exploiting server-side parameter pollution in a query string

I sent the `administrator%26test` payload. I checked the error message to understand how the server handled extra parameters.

- Injection

```
administrator%26test -> "error": "Parameter is not supported."
```

<img src="/assets/posts/apitesting/image (3).png" width="700" >

I sent the `administrator%26username` payload. The server returned the user's email. This confirmed that `username` is a valid parameter.

- Injection

```
administrator%26username -> {"result":"*****@normal-user.net","type":"email"}
```

<img src="/assets/posts/apitesting/image (4).png" width="700" >

I tested the `administrator%23` payload. I checked the response to understand how the `#` character affected the request.

- Injection → Parameter Pollution

```
administrator%23  -> "error": "Field not specified."
```

<img src="/assets/posts/apitesting/image (5).png" width="700" >

I sent the `administrator%26field=test%23` payload. I analyzed the parameter validation from the error message.

- Injection

```
administrator%26field=test%23 -> "error":"Invalid field."
```

<img src="/assets/posts/apitesting/image (6).png" width="700" >

I used Burp Intruder to test different parameter names. I compared the server responses.

<img src="/assets/posts/apitesting/image (7).png" width="700" >

I analyzed the results and found two valid parameters: **username** and **email**.

<img src="/assets/posts/apitesting/image (8).png" width="700" >

I checked the JavaScript file and found the `reset_token` parameter.

<img src="/assets/posts/apitesting/image (9).png" width="700" >

I added the `reset_token` parameter using Server-Side Parameter Pollution. The server returned the password reset link.

<img src="/assets/posts/apitesting/image (10).png" width="700" >

I opened the password reset link. I set a new password and solved the lab.

<img src="/assets/posts/apitesting/image (11).png" width="700" >


### 03. Finding and exploiting an unused API endpoint

I captured the product request in Burp Suite. I checked the response and found the API endpoint.

<img src="/assets/posts/apitesting/image (12).png" width="700" >

I sent a **POST** request. The server returned **Method Not Allowed** and showed the allowed methods.

<img src="/assets/posts/apitesting/image (13).png" width="700" >

I sent a **PATCH** request. The response showed that only **application/json** is supported.

<img src="/assets/posts/apitesting/image (14).png" width="700" >

I changed the request to **application/json**. I checked the new response from the server.

<img src="/assets/posts/apitesting/image (15).png" width="700" >

I sent an empty JSON body. The server said the **price** parameter was missing.

<img src="/assets/posts/apitesting/image (16).png" width="700" >

I sent the `price` parameter as text. The server expected a valid non-negative number.

<img src="/assets/posts/apitesting/image (17).png" width="700" >

I sent the `price` parameter as a number. The API updated the product price successfully.

<img src="/assets/posts/apitesting/image (18).png" width="700" >

I refreshed the product page. The new price was shown on the website.

<img src="/assets/posts/apitesting/image (19).png" width="700" >

I changed the product price to **0**. The server accepted the request and returned the new price.

<img src="/assets/posts/apitesting/image (20).png" width="700" >

I bought the product with the new price. The order was completed and I solved the lab.

<img src="/assets/posts/apitesting/image (21).png" width="700" >


### 04. Exploiting a mass assignment vulnerability

I captured the checkout request in Burp Suite. The purchase failed because of insufficient funds. I checked the API response.

<img src="/assets/posts/apitesting/image (22).png" width="700" >

I checked the cart response. I found the `chosen_discount` object. This field looked useful for a Mass Assignment attack.

<img src="/assets/posts/apitesting/image (23).png" width="700" >

I added the `chosen_discount` parameter to the request. I tried to change the discount value. The server processed this field.

<img src="/assets/posts/apitesting/image (24).png" width="700" >

I sent a discount value of **200**. The server returned an error because the value was invalid.

<img src="/assets/posts/apitesting/image (25).png" width="700" >

I changed the discount value to **100**. The server accepted the request and created the order.

<img src="/assets/posts/apitesting/image (26).png" width="700" >

The order was completed successfully. I bought the product for free by using the Mass Assignment vulnerability. The lab was solved.


### 05. xploiting server-side parameter pollution in a REST URL

I tested the REST URL with the `%23` character. I checked the error message to understand how the server handled the URL.

<img src="/assets/posts/apitesting/image (27).png" width="700" >

I added `/` to the end of the URL. The server returned the user information. This confirmed that I could change the URL path.

<img src="/assets/posts/apitesting/image (28).png" width="700" >

I tested the API with `/../` path traversal. I analyzed the server responses.

<img src="/assets/posts/apitesting/image (29).png" width="700" >

I added more path traversal sequences. The server processed the request and returned different responses.

<img src="/assets/posts/apitesting/image (30).png" width="700" >

I accessed the `openapi.json` file. I found the API documentation and identified useful endpoints.

<img src="/assets/posts/apitesting/image (31).png" width="700" >

I checked the JavaScript file. I found that the application uses the `resetToken` parameter.

<img src="/assets/posts/apitesting/image (32).png" width="700" >

I used the endpoint from the API documentation. I added the `field=passwordResetToken` parameter and obtained the administrator's reset token.

<img src="/assets/posts/apitesting/image (33).png" width="700" >

- Injection

```
/api/internal/v1/users/administrator/../../../../../../../../api/internal/v1/users/administrator/field/passwordResetToken#/field/email
```

I combined the endpoint with path traversal. The request returned the password reset token.

<img src="/assets/posts/apitesting/image (34).png" width="700" >

I opened the password reset page. I set a new password and solved the lab.