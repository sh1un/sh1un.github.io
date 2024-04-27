---
title: 'Serverless Framework 101 - 輕鬆開發並快速部署你的 AWS Lambda'
date: 2024-04-28T04:27:23+08:00
draft: false
description: "學習如何透過 Serverless Framework 來開發並部署你的 AWS Lambda"
tags: ["AWS Lambda", "Serverless Framework", "Serverless"]
categories: ["AWS", "Serverless"]
keywords:
- AWS
- AWS Lambda
- Lambda
- SAM
- AWS SAM
- Serverless
- Serverless Framework
- 無服務器部署
- CloudWatch Logs
- Python
---
## Serverless Framework 簡介

Serverless Framework 是一個開源的無服務器應用框架，它允許開發者快速建立、部署和管理在 AWS Lambda、Google Cloud Functions、Azure Functions 等雲平台上運行的無服務器應用。這個框架使用一個簡潔的配置文件（通常是 `serverless.yml`），在其中定義了應用的所有資源和設定，讓開發者可以專注於編寫業務邏輯而非管理基礎設施。

---

## 安裝 Serverless Framework

**Prerequisites:**

- 需要有 npm，若你的電腦沒有 npm，請去下載 [NodeJS](https://nodejs.org/en)

```bash
$ npm install -g serverless
```

---

## 創建一個 Service

```bash

$ serverless

? What do you want to make?
  AWS - Node.js - Starter
  AWS - Node.js - HTTP API
  AWS - Node.js - Scheduled Task
  AWS - Node.js - SQS Worker
  AWS - Node.js - Express API
  AWS - Node.js - Express API with DynamoDB
  AWS - Python - Starter
> AWS - Python - HTTP API
  AWS - Python - Scheduled Task
  AWS - Python - SQS Worker
  AWS - Python - Flask API
  AWS - Python - Flask API with DynamoDB
  Other
```

輸入指令後透過方向鍵選取你要的 Template，本文將已 AWS - Python HTTP API 示範

如果這些模板沒有你滿意的，可以到 [serverless/examples](https://github.com/serverless/examples) 找你要的模板，然後指令輸入 `serverless --template-url=https://github.com/serverless/examples/tree/v3/...`

```bash
? What do you want to call this project? serverless-framework-101

✔ Project successfully created in serverless-framework-101 folder

? Register or Login to Serverless Framework Yes
Logging into the Serverless Framework via the browser
If your browser does not open automatically, please open this URL:
https://app.serverless.com?client=cli&transactionId=x6r7NmzVa-ExeVdCyQZV2

✔ You are now logged into the Serverless Framework

✔ Your project is ready to be deployed to Serverless Dashboard (org: "shiunchiu", app: "serverless-framework-101")
```

進入你的工作目錄，然後開啟 VS Code

```bash
$ cd your-service-name
$ code .
```

現在的目錄架構應該會長這樣

```bash
C:.
    .gitignore
    handler.py
    README.md
    serverless.yml
```

---

## 配置 Provider

### 登入 AWS 帳號

請先[登入你的 AWS 帳號](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fconsole.aws.amazon.com%2Fconsole%2Fhome%3FhashArgs%3D%2523%26isauthcode%3Dtrue%26nc2%3Dh_ct%26src%3Dheader-signin%26state%3DhashArgsFromTB_ap-southeast-2_19e02f31a8db0977&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fcanvas&forceMobileApp=0&code_challenge=ptcq9Yh1TsOUWf-Kpzcitdm1y-90fE2tuRRHKxZCj-4&code_challenge_method=SHA-256)

### 到 Serverless Framework Dashboard 配置 Provider

![Untitled](https://github.com/sh1un/sh1un.github.io/assets/85695943/8447f951-149f-4b92-af94-731b48e3f538)

![Untitled 1](https://github.com/sh1un/sh1un.github.io/assets/85695943/62345dba-ddcf-4acd-aab3-78f840711d1c)

按下 Connect AWS Provider 後，會跳轉到 AWS CloudFormation 頁面，直接下滑到底，打勾 “**I acknowledge that AWS CloudFormation might create IAM resources with custom names.**” → Create Stack
![Untitled 2](https://github.com/sh1un/sh1un.github.io/assets/85695943/b4d432f5-1efb-4ab0-867a-1b62e5e16158)

等待一下，等 Stack 創建好我們可以去看一下這個 Stack，其實他就是幫你自動創建一個 IAM Role
![Untitled 3](https://github.com/sh1un/sh1un.github.io/assets/85695943/1f1af2da-64cb-4858-ad1d-0b13ba418f40)

Stack 創建好後，跳轉到 Serverless Framework Dashboard，就會看到創建好的 Provider

![Untitled 4](https://github.com/sh1un/sh1un.github.io/assets/85695943/d58792bd-166b-4997-be89-cb9d14db908d)

---

## 寫好 Code，把你的 Code 部署到 AWS

```bash
$ serverless deploy # or sls deploy

Deploying serverless-framework-101 to stage dev (us-east-1, "serverless-framework-101-dev" provider)
Warning: Serverless Framework observability features do not support the following runtime: python3.11
✔ Serverless Framework's Observability features being set up on your AWS account (one-time set-up).
  An email will be sent upon completion, or view progress within the Dashboard:
  https://app.serverless.com/shiunchiu/settings/integrations
✔ Serverless Framework Observability is enabled

✔ Service deployed to stack serverless-framework-101-dev (173s)

dashboard: https://app.serverless.com/shiunchiu/apps/serverless-framework-101/serverless-framework-101/dev/us-east-1
endpoint: GET - https://1ibju7jfc9.execute-api.us-east-1.amazonaws.com/
functions:
  hello: serverless-framework-101-dev-hello (2 kB)
```

> 如果比較懶想使用 sls deploy 指令，但你又使用 Powershell ，有可能會如果看到這種輸出，這是因為 PowerShell 將 **`sls`** 誤認為是 **`Select-String`** 的別名，這是 PowerShell 中用於搜索字符串的內建命令。建議換個 git bash 或是就乖乖打完整指令
>
> ```bash
> $ sls deploy
>
> cmdlet Select-String at command pipeline position 1
> Supply values for the following parameters:
> Path[0]:
> ```

### 到 AWS Console 查看部署結果

我們可以先到 Lambda 頁面，可以看到部署上去的 function

![Untitled 5](https://github.com/sh1un/sh1un.github.io/assets/85695943/9200f546-028f-466f-8ac5-524378667d5a)

因為 Serverless Framework 其實是透過 CloudFormation 來部署，現在我們到 CloudFormation 頁面查看，也會看到有新的 Stack

![Untitled 6](https://github.com/sh1un/sh1un.github.io/assets/85695943/a1ce9ca5-77f0-4df6-90d6-877e3fc43898)

---

## 調用 Lambda Function

當我們部署好之後，就可以來調用看看!

```bash
$ serverless invoke -f hello

{
    "statusCode": 200,
    "body": "{\"message\": \"Go Serverless v3.0! Your function executed successfully!\", \"input\": {}}"
}
```

透過 `serverless invoke` 指令就可以調用我們部署上去的 Lambda

但需要特別注意 `-f` 後面的值是要根據你在 `serverless.yml` 上面所配置的 function name 來設定，並非已經部署上去的 Lambda Function name

現在我們上去 CloudWatch 看 Log，會看到我們剛剛的調用

![Untitled 7](https://github.com/sh1un/sh1un.github.io/assets/85695943/f9936abd-17dc-4505-8f0a-f999057a1e74)

有時候我們會想要直接在 Local 看到 log，那可以剛剛的 `serverless invoke` 指令加上 `--log` ，這樣就不用到 AWS Console 查看了

```bash
$ serverless invoke -f hello --log

{
    "statusCode": 200,
    "body": "{\"message\": \"Go Serverless v3.0! Your function executed successfully!\", \"input\": {}}"
}
--------------------------------------------------------------------
START
SERVERLESS_TELEMETRY.TZ.H4sIAPFVLWYC/43QMUoDQRTGcSO4wpYBEbcxhIAYmGVmZ+bNTDoFG7EzlSA6+2aWhGyyYTduIuIhvELO4B0EC4/gBSy8glELLYTYveLBx+8f1mEHEETmPRLDJCUCHRAjE0pMmjqQjnKPNmpXvqx9mfuqIllpx35elCPCKCPO12Tg87zodsO9X192XpHcjlNnSeVGzS0a8ziJguntbFBMoqdG2NKWKp6i4AoFGOtQayW4BKe4FYkRzVaK4JGjlhI48NRpzsAw7TMJic5k1g5XK/H3yuHB69v+Y+95l928vH8dPVwG18vgMgwWGq5AHP8DcdqRzBoEikQ50ESwhBNNVzkSoaSXVFum+MV25+yof3Lenzaih08JNZDwjHMljKCrmg5TipkU1BrnwKyXRGtrtHd+rPFwUhdoZ8Ni8gd7o7t5d/8BvrD25toBAAA=
END Duration: 4.13 ms Memory Used: 57 MB
```

---

## Stream Logs

其實我們還可以讓我們的 Log 在 Terminal 串流，我會在 VS Code 開啟兩個 Terminal 來示範

在 VS Code 打開一個 Terminal ( 快捷鍵: Ctrl + `)

輸入以下指令

```bash
$ serverless logs -f hello --tail
```

我們再接著建立一個新的 Terminal

![Untitled 8](https://github.com/sh1un/sh1un.github.io/assets/85695943/d80b1691-0134-47f1-ad2d-fd9d2037da74)

配置一下版面，依照下圖拖曳 Terminal 到上面

![Untitled 9](https://github.com/sh1un/sh1un.github.io/assets/85695943/3854409b-009f-41e1-9baf-8fea47113a06)

然後使用下方的 Terminal 來調用 Lambda Function

```bash
$ serverless invoke -f hello

{
    "statusCode": 200,
    "body": "{\"message\": \"Go Serverless v3.0! Your function executed successfully!\", \"input\": {}}"
}
```

直接看下方的 gif 圖感受一下吧！ (這 gif 圖長達 27s 敬請耐心等待)

![ServerlessFramework101LambdaStreamLogs](https://github.com/sh1un/sh1un.github.io/assets/85695943/99bec5de-f18a-44ca-a07b-d0460cab396d)

---

## 參考資料

- [Serverless Framework - Setting Up Serverless Framework With AWS](https://www.serverless.com/framework/docs/getting-started)
- [Serverless Framework - Providers](https://www.serverless.com/framework/docs/guides/providers)
- [GitHub - serverless/examples](https://github.com/serverless/examples)
