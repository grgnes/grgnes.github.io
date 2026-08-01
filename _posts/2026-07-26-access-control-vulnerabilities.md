---
layout: post
title: "Access Control Vulnerabilities"
date: 2026-07-26

category: PortSwigger

image: /assets/posts/access-control-vulnerabilities/cover.png

excerpt: "PortSwigger Web Security Academy Access Control labs write-up with practical notes and exploitation steps."

tags:
- PortSwigger
- Web Security
- Access Control
- Authorization
---

## Introduction

Access control vulnerabilities happen when users can access pages, data, or functions they should not be able to access.

---

## Labs

### 01. Unprotected Admin Functionality

When the website path is scanned with ffuf tool, open paths are found.

```bash
ffuf -u https://0a8b00cb03125ee78240ba180006002c.web-security-academy.net/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt 
```
<img src="/assets/posts/access-control-vulnerabilities/01.png" width="700" >

There are some opened paths. Robots.txt can checked or when looking at the administrator-panel path, the user list can be found.

<img src="/assets/posts/access-control-vulnerabilities/02.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/03.png" width="700" >

### 02. Unprotected Admin Functionality with Unpredictable URL

A hidden website path was found when checking the developer tools.

<img src="/assets/posts/access-control-vulnerabilities/04.png" width="600" >

When looking at that website path, the users were listed.

<img src="/assets/posts/access-control-vulnerabilities/05.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/06.png" width="700" >

### 03. User Role Controlled by Request Parameter

When logging in with the provided credentials, there is a cookie about admin info at the response.

<img src="/assets/posts/access-control-vulnerabilities/07.png" width="700" >

when looking the cookie info from developer tools, there is a cookie named “Admin” and its value is false.

<img src="/assets/posts/access-control-vulnerabilities/08.png" width="700" >

If the value is changed to true, the /admin panel can be accessed.

<img src="/assets/posts/access-control-vulnerabilities/09.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/10.png" width="700" >


### 04. User Role Can Be Modified in User Profile

When logging in with the provided credentials and enter your email address, the response contains extra information besides email: the username, Apikey and roleid.

<img src="/assets/posts/access-control-vulnerabilities/11.png" width="700" >

The request is sent to the repeater,missing information is added and the roleid value changed to 2, the admin panel button appears.

<img src="/assets/posts/access-control-vulnerabilities/12.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/13.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/14.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/15.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/16.png" width="700" >


### 05. User ID Controlled by Request Parameter

When logging in using the provided credentials, the API key information is gived.

<img src="/assets/posts/access-control-vulnerabilities/17.png" width="700" >

And checking the URL, there is a id parametrer named wiener.

<img src="/assets/posts/access-control-vulnerabilities/18.png" width="700" >

Then, id parametrer name is changed to Carlos, it gives carlos’ apikey information.

<img src="/assets/posts/access-control-vulnerabilities/19.png" width="700" >

### 06. User ID Controlled by Request Parameter with Unpredictable User IDs

First finding a blog post by carlos and click username the Carlos.

<img src="/assets/posts/access-control-vulnerabilities/20.png" width="700" >

When looking the request, there is a userId about Carlos. 

<img src="/assets/posts/access-control-vulnerabilities/21.png" width="700" >

```bash
GET /blogs?userId=0d84ad23-c0b1-4183-a45a-598b39a8dbb0 
```

When logging in using the provided credentials, the API key information is gived about Wiener. At the URL, there is a id about Wiener.

<img src="/assets/posts/access-control-vulnerabilities/22.png" width="700" >

When the id is changed to carlos’ userId, it gives carlos’ apikey.

<img src="/assets/posts/access-control-vulnerabilities/23.png" width="700" >

### 07. User ID Controlled by Request Parameter with Data Leakage in Redirect

When logging in using the provided credentials, there is a id parameter named wiener at the request. 

<img src="/assets/posts/access-control-vulnerabilities/24.png" width="700" >

Then, send to repeater and the id parameter is changed to carlos. It gives carlos’ apikey.

<img src="/assets/posts/access-control-vulnerabilities/25.png" width="700" >

### 08. User ID Controlled by Request Parameter with Password Disclosure

When logging in using the provided credentials, there is a id parameter named wiener at the request.

<img src="/assets/posts/access-control-vulnerabilities/26.png" width="700" >

Then, send to repeater the request and the id parameter is changed to administrator. It gives administator’s password on the screen.

<img src="/assets/posts/access-control-vulnerabilities/27.png" width="700" >

When logged in with an administrator account, an admin panel button is available. Clicking it lists the users, and the 'carlos' account is deleted.

<img src="/assets/posts/access-control-vulnerabilities/28.png" width="700" >

Clicking it lists the users, and the 'carlos' account is deleted.

<img src="/assets/posts/access-control-vulnerabilities/29.png" width="700" >

### 09. Insecure Direct Object References (IDOR)

When sending a message and view transcript, it downloaded a .txt file.

<img src="/assets/posts/access-control-vulnerabilities/30.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/31.png" width="700" >

The request is sent to the repeater, and the filename is changed to "1".

<img src="/assets/posts/access-control-vulnerabilities/32.png" width="700" >


### 10. URL-Based Access Control Can Be Circumvented

When clicking on the admin panel button, it says “access denied”

<img src="/assets/posts/access-control-vulnerabilities/33.png" width="700" >

In some server configurations, the application may take the X-Original-URL header into account instead of the actual URL and process the request as /admin. When the `X-Original-URL` header is added with /admin, the server returns a 200 status code instead of a 403, and the admin panel opens.

<img src="/assets/posts/access-control-vulnerabilities/34.png" width="700" >

To delete a user, navigate to the /delete path and delete the user named 'carlos'.

<img src="/assets/posts/access-control-vulnerabilities/35.png" width="700" >

When checking for the username "carlos," the page does not open.

<img src="/assets/posts/access-control-vulnerabilities/36.png" width="700" >


### 11. Method-Based Access Control Can Be Circumvented

First, log in as administrator. Then upgrade Carlos to admin and send the request to Repeater.

<img src="/assets/posts/access-control-vulnerabilities/37.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/38.png" width="700" >

Then, log in as wiener and copy wiener's session cookie.

<img src="/assets/posts/access-control-vulnerabilities/39.png" width="700" >

After replacing the cookie with wiener's session cookie, the server responds with "Unauthorized".

<img src="/assets/posts/access-control-vulnerabilities/40.png" width="700" >


If you use "Change request method" to change the request from POST to GET, and change the username to wiener, the lab is solved.

<img src="/assets/posts/access-control-vulnerabilities/41.png" width="700" >

The vulnerability is caused by HTTP method-based access control. The application checks permissions for POST requests but does not properly check GET requests.

### 12. Multi-Step Process with No Access Control on One Step

First, log in as administrator. Then upgrade Carlos to admin. 

<img src="/assets/posts/access-control-vulnerabilities/42.png" width="700" >

When the "Are you sure?" message appears, click Yes and send the request to Repeater.

<img src="/assets/posts/access-control-vulnerabilities/43.png" width="700" >

<img src="/assets/posts/access-control-vulnerabilities/44.png" width="700" >

Then, log in as wiener and copy wiener's session cookie.

<img src="/assets/posts/access-control-vulnerabilities/45.png" width="700" >

In the request sent to Repeater, replace Carlos's session cookie with wiener's session cookie, and change the username to wiener.

<img src="/assets/posts/access-control-vulnerabilities/46.png" width="700" >

The vulnerability is caused by missing access control in the second step of a multi-step process. The first step is protected, but the confirmation step can be accessed without proper authorization.

### 13. Referer-Based Access Control

First, log in as administrator. Then upgrade Carlos to admin and send the request to Repeater.

<img src="/assets/posts/access-control-vulnerabilities/47.png" width="700" >

Then, log in as wiener and copy wiener's session cookie from the developer tools.

<img src="/assets/posts/access-control-vulnerabilities/48.png" width="700" >

In the request sent to Repeater, replace Carlos's session cookie with wiener's session cookie, and change the username to wiener.

<img src="/assets/posts/access-control-vulnerabilities/49.png" width="700" >

The vulnerability is caused by trusting the Referer header. The application uses the Referer header for access control instead of checking the user's authorization.