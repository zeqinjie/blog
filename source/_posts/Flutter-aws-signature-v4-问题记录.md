---
title: Flutter-aws-signature-v4-问题记录
date: 2023-01-08 21:13:17
categories: "Flutter"
tags:
	- Flutter
---

## 前言

近期 Flutter IM 的模块优化，需要将素材生成 AWS Signature v4签名以上传到s3。后端新增两个接口，一个是预上传的接口获取上传 aws 相关参数，及获取签名需要 session_token 的接口。做了下遇到问题的笔记



## 遇到问题

### 预上传

 - 在预上传接口获取到 aws 需要的字段，原先封装的表单上传但却抛异常如下:

`flutter: DioError [DioErrorType.other]: HttpException: Connection closed while receiving data, uri = https://xxxxxxx.s3.ap-northeast-1.amazonaws.com` 

- 经排查发现是编码问题调用了`MultipartFile.fromString('$value');`  在 dio 默认是使用 UTF-8 编码了

```
  /// Creates a new [MultipartFile] from a string.
  ///
  /// The encoding to use when translating [value] into bytes is taken from
  /// [contentType] if it has a charset set. Otherwise, it defaults to UTF-8.
  /// [contentType] currently defaults to `text/plain; charset=utf-8`, but in
  /// the future may be inferred from [filename].
  factory MultipartFile.fromString(
    String value, {
    String? filename,
    MediaType? contentType,
    final Map<String, List<String>>? headers,
  }) {
    contentType ??= MediaType('text', 'plain');
    var encoding = encodingForCharset(contentType.parameters['charset'], utf8);
    contentType = contentType.change(parameters: {'charset': encoding.name});

    return MultipartFile.fromBytes(
      encoding.encode(value),
      filename: filename,
      contentType: contentType,
      headers: headers,
    );
  }
```

- 封装的文件上传方法如下

```dart
/// 上传文件
  Future<dynamic> uploadFile(
    String url,
    String filePath, {
    String customFileParamKey = 'file',
    Map<String, dynamic>? params,
  }) async {
    var file = File(filePath);
    if (!file.existsSync()) {
      return TWApiResponse(message: '文件不存在');
    }
    var map = <String, dynamic>{};
    map[customFileParamKey] = await MultipartFile.fromFile(filePath);
    if (params != null) {
      params.forEach((key, value) {
				map[key] = value; 
      });
    }
    FormData formData = FormData.fromMap(map);
    var options = Options(
      contentType: 'multipart/form-data;boundary=${formData.boundary}',
    );
    try {
      final response = (await client.post(
        url,
        options: options,
        data: formData,
      ));
      final code = response.statusCode;
      TWLog('statusCode: ${response.statusCode}');
      if (code == 204) {
        return TWApiResponse(code: TWChatResultCode.success.value);
      }
      return TWApiResponse(message: '上傳失敗');
    } catch (e) {
      debugPrint(e.toString());
      return TWApiResponse(message: '上傳失敗');
    }
  }
```

### v4 签名 URL

在 Flutter 端对 URL 做预签名，查阅官方文档后及 Flutter SDK 后，在 issue 找到提问 [How to use Amazon STS to upload/down file with amplify-flutter](https://github.com/aws-amplify/amplify-flutter/issues/2038)  参考 demo。 注意不同 demo 地方

 1、queryParameters 补充：X-Amz-Content-Sha256: UNSIGNED-PAYLOAD。

 2、AWSCredentials 初始化需传 expiration

- 封装方法如下

```dar
 /// 获取签名后的 url
  Future<String> fetchSignatureUrl({
    required String fileName,
    required String region,
    required String bucket,
    required String sessionToken,
    required String secretAccessKey,
    required String accessKeyId,
    required String expiration,
    int minutes = 55,
  }) async {
    try {
      final credentials = AWSCredentials(
        accessKeyId,
        secretAccessKey,
        sessionToken,
        expiration.getDateTime,
      );
      final signer = AWSSigV4Signer(
        credentialsProvider: AWSCredentialsProvider(credentials),
      );

      // Set up S3 values
      final scope = AWSCredentialScope(
        region: region,
        service: AWSService.s3,
      );
      final host = '$bucket.s3.$region.amazonaws.com';
      final serviceConfiguration = S3ServiceConfiguration();

      // Create a pre-signed URL for downloading the file
      final urlRequest = AWSHttpRequest.get(
        Uri.https(
          host,
          fileName,
          {
            AWSHeaders.contentSHA256:
                S3ServiceConfiguration.unsignedPayloadHash,
          },
        ),
        headers: {
          AWSHeaders.host: host,
        },
      );
      final signedUrl = await signer.presign(
        urlRequest,
        credentialScope: scope,
        serviceConfiguration: serviceConfiguration,
        expiresIn: Duration(minutes: minutes),
      );
      TWLog('Download URL: $signedUrl');
      return signedUrl.toString();
    } catch (e) {
      TWLog('fetchSignatureUrl ===> $e');
      return '$e';
    }
  }
```



