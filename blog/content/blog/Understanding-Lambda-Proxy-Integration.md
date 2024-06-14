---
title: '深度解析 AWS API Gateway 的 Lambda Proxy Integration 功能'
date: 2024-06-14T23:41:33+08:00
draft: false
description: "API Gateway 背後接 Lambda 是一個相當常見的組合，然而很多人對 API Gateway 上面的 Lambda Proxy Integration 在做什麼其實不是很了解，事實上，啟用或不啟用這個功能，行為會差非常多，這篇文章用實際的例子帶你認識 Lambda Proxy Integration"
tags: ["AWS", "AWS API Gateway", "AWS Lambda", "Lambda Proxy Integration", "Cloud Computing", "Serverless", "Backend Development", "HTTP Request"]
categories: ["AWS", "Serverless"]
keywords:
- AWS Lambda Proxy Integration
- API Gateway Lambda proxy integration
- API Gateway Lambda integration
- Lambda function HTTP request
- Serverless architecture
- AWS API Gateway setup
- Lambda Proxy advantages
- Lambda Proxy HTTP response
- AWS cloud services
- Simplify Lambda requests
- Lambda function event object
---

在 AWS API Gateway (REST API) 創建 method 時，我們可以指定他要把請求傳到哪個 AWS 服務，而如果你選擇 Lambda，你會看到有一個選項是 Lambda Proxy Integration，那我把它勾選起來會怎樣呢？

![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/95b1fe3d-a93c-4563-80bb-36ad8ff3b6c8)

## TL;DR

如果有打開 Proxy Integration，接著我們到那個 Resource 的 Method 頁面看一下，會看到下圖的提示
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/47c6b110-9431-4181-b584-9ab14db865c5)

> **Proxy integration**
> For proxy integrations, API Gateway automatically passes the backend output to the caller as the complete payload.

但也許沒有很好理解，所以我們後面會實際打 Code 來示範，但我這邊先稍微解釋一下打開這個功能的影響:

1. **完整的 HTTP 請求資訊**：啟用 Lambda Proxy Integration 後，API Gateway 會將整個 HTTP 請求打包成單一的 [event object](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format)，並傳遞給 Lambda Function
2. **簡化的處理流程**：Lambda Function 接收到的是一個包含了所有請求資訊的 event object，因此不需要再通過模板或轉換就可以直接處理請求。這樣簡化了函數的輸入處理，因為 Lambda Function 已經包含了所有必要的請求資訊
3. **控制 HTTP Response**：Lambda Function 需要返回一個特定格式的物件，這個物件包括 Status Code、Response Headers 和 Response Body。由於 Response 是直接由 Lambda Function 生成的，所以開發者可以完全控制 Response 的格式和內容
4. **效能和成本效益**：由於請求處理的過程更加直接，減少了中間的處理和轉換步驟，可以提高處理效率，可能也會在某種程度上降低成本

總之，選擇 Lambda Proxy Integration 可以使得與 AWS Lambda 的整合更加緊密、直接和高效。這適合需要精細控制 HTTP 響應的場景，或者希望簡化 API 與 Lambda 之間交互的架構設計

## 來看實際的例子先來感受有無開啟的行為差異

我現在創建了兩個 API Gateway -> Lambda 的**組合**:

1. 使用 Proxy Integration
2. 不使用 Proxy Integration

而程式碼目前都是:

```python
import json

def lambda_handler(event, context):
    # TODO implement
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }

```

接著我們分別執行看看

- With Lambda Proxy Integration

```shell
# With Lambda Proxy Integration
$ curl -X GET "https://m13nrn7ks1.execute-api.us-west-2.amazonaws.com/dev"

"Hello from Lambda!"

```

- Non Lambda Proxy Integration

```shell
# Non Lambda Proxy Integration
$ curl -X GET https://byjlq9pv48.execute-api.us-west-2.amazonaws.com/dev/

{"statusCode": 200, "body": "\"Hello from Lambda!\""}

```

可以注意到，回傳的格式不太一樣，前者是回傳單純的字串，後者才是包在 Json 物件內而且還多了 `statusCode` 在 Response Body，具體原因後面會解釋

### 修改 Lambda Function 內 Return 的 Status Code

我們緊接著來改一下 Lambda Function 的程式碼，我把 `statusCode` 改成 `400`，所以現在的完整程式碼如下:

```python
import json

def lambda_handler(event, context):
    # TODO implement
    return {
        'statusCode': 400, # 修改這行
        'body': json.dumps('Hello from Lambda!')
    }

```

一樣我依序調用 API，但是我這邊改成用 Postman 去調用，這樣比較好閱讀和觀覽，請幫我注意下面兩張圖片的紅框處:

- With Lambda Proxy Integration
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/d433c8cb-a2be-4b58-8849-9075d52ef766)

- Non Lambda Proxy Integration
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/2777e74c-c5d0-4d38-99f7-20810b398e43)

注意到了嗎？有沒有打開 Lambda Proxy Integration 其行為差異會如此大

### 加入 Query Parameters

接著我現在要在 API 調用時，加上 Query Parameter `name=Shiun`，並且我也修改了程式碼:

```python
import json

def lambda_handler(event, context):
    name = event["queryStringParameters"]["name"] # 從 evnet object 取得 Query Parameter
    return {
        'statusCode': 200, # 改回 200
        'body': json.dumps(f'Hello {name}!')
    }

```

一樣我再次分別調用兩支 API

- With Lambda Proxy Integration
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/89f50a1a-75c4-4b59-8b18-bb674e283a71)

- Non Lambda Proxy Integration
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/7f6b789e-df36-42d3-a664-4c6b04da1cf2)

可以注意到，有使用 Lambda Proxy Integration 的正確接到值了，而另一個沒有開啟的則出現錯誤，錯誤很明顯就是他找不到對應的 Key

### 加入自定義 Header

接下來我要在Lambda Function 中 return 自定義的 Header: `X-Shiun-Custom-Header" : "test"`
也是一樣，我會分別調用這兩支 API，我們來觀察 Response Headers

- With Lambda Proxy Integration
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/7a895fae-88fe-450f-8feb-1ade4a1bf0ce)

- Non Lambda Proxy Integration
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/df22fa12-9acb-4faf-af98-d77dc5f9910e)

觀察到兩者的差異了嗎？在 Lambda Proxy Integration 那張圖片看到 Response Headers 裡面真的有回傳 `X-Shiun-Custom-Header" : "test"`

## Lambda Proxy Integration 的工作原理

啟用 Lambda Proxy Integration 後，API Gateway 會將整個 HTTP 請求(包括request headers, query string parameters, URL path variables, payload, and API configuration data) 打包成單一的 [event object](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format)，並傳遞給 Lambda 函數。這也解釋為什麼我 call API 如果帶了 Query Param ，我使用這種寫法 `name = event["queryStringParameters"]["name"]` 能抓到 API 請求的 Query Parameter

而這個 event object 實際長這樣:

```json
{
  "resource": "/my/path",
  "path": "/my/path",
  "httpMethod": "GET",
  "headers": {
    "header1": "value1",
    "header2": "value1,value2"
  },
  "multiValueHeaders": {
    "header1": [
      "value1"
    ],
    "header2": [
      "value1",
      "value2"
    ]
  },
  "queryStringParameters": {
    "parameter1": "value1,value2",
    "parameter2": "value"
  },
  "multiValueQueryStringParameters": {
    "parameter1": [
      "value1",
      "value2"
    ],
    "parameter2": [
      "value"
    ]
  },
  "requestContext": {
    "accountId": "123456789012",
    "apiId": "id",
    "authorizer": {
      "claims": null,
      "scopes": null
    },
    "domainName": "id.execute-api.us-east-1.amazonaws.com",
    "domainPrefix": "id",
    "extendedRequestId": "request-id",
    "httpMethod": "GET",
    "identity": {
      "accessKey": null,
      "accountId": null,
      "caller": null,
      "cognitoAuthenticationProvider": null,
      "cognitoAuthenticationType": null,
      "cognitoIdentityId": null,
      "cognitoIdentityPoolId": null,
      "principalOrgId": null,
      "sourceIp": "IP",
      "user": null,
      "userAgent": "user-agent",
      "userArn": null,
      "clientCert": {
        "clientCertPem": "CERT_CONTENT",
        "subjectDN": "www.example.com",
        "issuerDN": "Example issuer",
        "serialNumber": "a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1:a1",
        "validity": {
          "notBefore": "May 28 12:30:02 2019 GMT",
          "notAfter": "Aug  5 09:36:04 2021 GMT"
        }
      }
    },
    "path": "/my/path",
    "protocol": "HTTP/1.1",
    "requestId": "id=",
    "requestTime": "04/Mar/2020:19:15:17 +0000",
    "requestTimeEpoch": 1583349317135,
    "resourceId": null,
    "resourcePath": "/my/path",
    "stage": "$default"
  },
  "pathParameters": null,
  "stageVariables": null,
  "body": "Hello from Lambda!",
  "isBase64Encoded": false
}
```

此外，啟用 Lambda Proxy Integration，你也必須遵守特定的 Output Format，你才能正確地把 Response 傳遞出去:

```json
{
    "isBase64Encoded": true|false,
    "statusCode": httpStatusCode,
    "headers": { "headerName": "headerValue", ... },
    "multiValueHeaders": { "headerName": ["headerValue", "headerValue2", ...], ... },
    "body": "..."
}
```

所以這邊也解釋了，為什麼設定好 `statusCode`、`headers`，我就能控制整個 Response 回應的 Status Code 和 Headers，這中間都是因為有了 Lambda Proxy Integration 幫我們簡化了這中間繁雜的配置，如果沒有 Lambda Proxy Integration，那我們就必須手動去 API Gateway 那邊設定 Mapping Template，配置起來相當麻煩。

## 那我到底要不要打開 Lambda Proxy Integration?

我會高度推薦打開，因為你可以在程式碼那邊控制 Response，這我不認為不只是是”簡化”，而是”大大降低管理複雜度”，我認為這是一個更佳的管理方式 (這邊是我的主觀感受，並不代表完全正確)。

甚至有時候我都會直接果斷啟用 Lambda Proxy Integration，然後在 API Gateway 後面加一層 Lambda 專門做參數驗證再轉發到實際後端的 Lambda Function，而非選擇在 API Gateway 那邊做請求預處理或是參數驗證，但實際仍需要以本身的業務需求去挑出最佳的解決方案，這邊只是提供一個我認為對開發人員很友善的處理方式，而且這種方式你也可以在程式碼內做出高度客製化

總而言之，如果你沒有使用 Proxy Integration，則需要在 API Gateway 中配置額外的轉換模板來處理輸入和輸出格式。

## Resources

- [Using AWS API Gateway as Proxy to other HTTP Endpoints](https://medium.com/@chikim79/using-aws-api-gateway-as-proxy-to-another-http-endpoints-5271d78c5bd6)
- [Set up Lambda proxy integrations in API Gateway - Amazon API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format)
