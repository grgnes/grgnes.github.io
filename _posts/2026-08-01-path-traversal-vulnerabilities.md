---
layout: post
title: "Path Traversal Vulnerabilities"
date: 2026-08-01

category: PortSwigger

image: /assets/posts/access-control-vulnerabilities/cover.png

excerpt: "PortSwigger Web Security Academy Path Traversal labs write-up with practical notes and exploitation steps."

tags:
- PortSwigger
- Web Security
- Path Traversal
---

## Introduction

Path Traversal (also known as Directory Traversal) is a vulnerability that allows an attacker to read files on the server that they should not be able to access.

## Cheat Sheet 
Check out the related Path Traversal Cheat Sheet:
**[Path Traversal Cheat Sheet](/cheat-sheets/#path-traversal)**


## Labs

### 01. File path traversal, simple case

I turned on intercept, clicked one of product and i captured the request. After forwarding it, i noticed a parameter named filename.

<img src="/assets/posts/path-traversal-vulnerabilities/image.png" width="700" >

I send the request to repeater.

<img src="/assets/posts/path-traversal-vulnerabilities/image (1).png" width="700" >

When i sent to the request, the response returned the contents of the requested image.

I wondered what would happen if I added `../` to the `filename` parameter.

<img src="/assets/posts/path-traversal-vulnerabilities/image (2).png" width="700" >

The response returned  “no such file”.  I kept adding more `../` sequences, but the server continued to return the same response.

Then i tried accessing a well-known system file by requesting `/etc/passwd`.

<img src="/assets/posts/path-traversal-vulnerabilities/image (3).png" width="700" >

This time, the server returned the contents of the `/etc/passwd` file, which contains the system's user account information. This confirmed that the application was vulnerable to path traversal, allowing access to files outside the intended directory.

### 02. File path traversal, traversal sequences blocked with absolute path bypass

Again i turned on intercept, clicked one of product and i captured the request. After forwarding it, i noticed a parameter named filename.

<img src="/assets/posts/path-traversal-vulnerabilities/image (4).png" width="700" >

I send the request to repeater and tried adding `../` to the `filename` parameter.

<img src="/assets/posts/path-traversal-vulnerabilities/image (5).png" width="700" >

The server returned the same **"No such file"** error. I then tried accessing a well-known system file by requesting `../etc/passwd`, but the server continued to return the same error.

<img src="/assets/posts/path-traversal-vulnerabilities/image (6).png" width="700" >

Since the traversal sequences appeared to be blocked, i tried using an absolute path instead.

This time, I requested `/etc/passwd` directly without using any `../` sequences.

<img src="/assets/posts/path-traversal-vulnerabilities/image (7).png" width="700" >

The application returned the contents of the `/etc/passwd` file.


### 03.File path traversal, traversal sequences stripped non-recursively

Again i turned on intercept, clicked one of product and i captured the request. After forwarding it, i noticed a parameter named filename.

<img src="/assets/posts/path-traversal-vulnerabilities/image (8).png" width="700" >

When i sent the request to repeater and first tried a standard path traversal payload using `../`. However, it did not work, indicating that the application was filtering traversal sequences.

<img src="/assets/posts/path-traversal-vulnerabilities/image (9).png" width="700" >

Next, I tried using a nested traversal sequence such as ....//, but the server still returned the "No such file" error.

<img src="/assets/posts/path-traversal-vulnerabilities/image (10).png" width="700" >

Finally, I increased the number of nested traversal sequences until the payload successfully reached the target file.

<img src="/assets/posts/path-traversal-vulnerabilities/image (11).png" width="700" >

- Injection

```bash
GET /image?filename=....//....//....//etc/passwd 
```

### 04. File path traversal, traversal sequences stripped with superfluous URL-decode

Again, I captured the request and sent it to repeater.

First, i tried a standard path traversal payload using `../`, but the application returned the same **"No such file"** error.

<img src="/assets/posts/path-traversal-vulnerabilities/image (12).png" width="700" >

Next, i URL-encoded the traversal payload. However, the application still returned the same error, indicating that the encoded payload was being filtered as well.

<img src="/assets/posts/path-traversal-vulnerabilities/image (13).png" width="700" >

Finally, i double URL-encoded the traversal sequences. This time, the application returned the contents of the `/etc/passwd` file.

<img src="/assets/posts/path-traversal-vulnerabilities/image (14).png" width="700" >

In this scenario, the web server filters path traversal sequences such as `../` after decoding the request. As a result, a normally URL-encoded payload is decoded first and then blocked by the filter. This is not always handled by the backend application alone. The request may be decoded or normalized by the web server or another intermediary before it reaches the application.


### 05. File path traversal, validation of start of path

I turned on Intercept, clicked on one of the products, and captured the request. After forwarding it, I noticed that the filename parameter contained the path /var/www/images/53.jpg.

I sent the request to Repeater and observed that the application expected the supplied filename to start with `/var/www/images/`. Simply requesting `/etc/passwd` would not satisfy this validation.

<img src="/assets/posts/path-traversal-vulnerabilities/image (15).png" width="700" >

Instead, I kept the required base path and appended traversal sequences to escape the directory:

```bash
/var/www/images/../../../etc/passwd
```

<img src="/assets/posts/path-traversal-vulnerabilities/image (16).png" width="700" >

The application accepted the request because the path still started with the expected directory. However, after resolving the traversal sequences, it accessed `/etc/passwd` and returned its contents.


### *06. ile path traversal, validation of file extension with null byte bypass

Again, I turned on Intercept, clicked on one of the products, and captured the request. After forwarding it, I noticed a parameter named `filename`.

I sent the request to Repeater and tried a standard path traversal payload to access `/etc/passwd`. However, the application returned "No such file" because it required the supplied filename to end with the `.png` extension.

<img src="/assets/posts/path-traversal-vulnerabilities/image (17).png" width="700" >

To bypass this validation, I appended a **URL-encoded null byte** (`%00`) before the required `.png` extension:

```bash
../../../etc/passwd%00.png
```

<img src="/assets/posts/path-traversal-vulnerabilities/image (18).png" width="700" >


## Source Code Examples

The vulnerable and secure backend examples used in this write-up are available on GitHub: 

**[https://github.com/grgnes/Path-Traversal-Vulnerabilities](https://github.com/grgnes/Path-Traversal-Vulnerabilities)**