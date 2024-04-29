---
title: 'Aws Sam 101'
date: 2024-04-29T23:47:09+08:00
draft: false
description: "最近在學習 AWS SAM，本文"
tags: ["", ""]
categories: [""]
keywords:
- 
---

# AWS SAM 101

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2011.png)

## Q&A

- AWS 尋找 Cred 的順序是什麼?
    
    [Using temporary credentials with AWS resources - AWS Identity and Access Management](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html#using-temp-creds-sdk-cli)
    
    AWS CLI 在尋找認證資料（Credentials）時，會按照一定的順序來搜尋，這個順序確保了不同設定之間的適當優先級。以下是 AWS CLI 尋找認證資料的順序：
    
    1. **命令列選項**：可以直接在 AWS CLI 命令中指定 `-profile` 來使用特定的配置文件中的認證，或使用 `-access-key`, `-secret-key`, 和 `-session-token` 來直接指定認證。
    2. **環境變數**：AWS CLI 會檢查是否設定了以下環境變數：
        - `AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`、`AWS_SESSION_TOKEN`（用於臨時認證）
    3. **CLI 配置檔案**：CLI 配置檔案通常位於 `~/.aws/credentials`（Linux 或 macOS）或 `C:\\Users\\USERNAME\\.aws\\credentials`（Windows）。您可以在這些檔案中配置一或多個配置文件。
    4. **CLI 配置檔案中的配置檔案**：除了認證檔案外，CLI 還會查看配置檔案（位於 `~/.aws/config`），這裡可以包含一個 `profile` 的預設區域和輸出格式設定。
    5. **容器認證**：如果您的應用運行在 Amazon ECS 上並使用了角色，AWS CLI 可以透過容器認證提供者來獲取認證。
    6. **EC2 實例角色**：如果您的應用運行在 EC2 實例上，且該實例具有 IAM 角色，則 AWS CLI 將使用該角色提供的認證。這是通過實例元數據服務來實現的。
    
    以上這些方式提供了靈活的配置選項，允許您根據不同的使用情景來選擇合適的認證配置方法。
    
- .reg 檔是什麼
    
    `.reg` 檔案是一種 Windows 註冊表檔案，用於在 Windows 註冊表中新增、修改或刪除註冊表項目。這些檔案是以純文字格式儲存，通常被用來快速配置系統設定或解決特定的系統問題。
    
    當你雙擊或通過命令行運行 `.reg` 檔案時，檔案中的設定將會自動應用到 Windows 註冊表中。這種檔案常被用於批量更新註冊表設定，便於用戶或系統管理員快速部署多台機器的註冊表變更。
    
    `.reg` 檔案內容通常包括以下幾個部分：
    
    - **Windows Registry Editor Version**：這是檔案的首行，指明檔案遵循的註冊表編輯器版本。
    - **註冊表路徑**：指定要更改的註冊表鍵的完整路徑。
    - **鍵值對**：設定的名稱和值，指定要添加或修改的具體註冊表設定。
    
    以下是一個簡單的 `.reg` 檔案的例子，用於設定長路徑行為：
    
    ```
    Windows Registry Editor Version 5.00
    
    [HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Control\\FileSystem]
    "LongPathsEnabled"=dword:00000001
    
    ```
    
    在這個例子中，`LongPathsEnabled` 設為 `1`，啟用了對長路徑的支持
    
    。這個設定告訴 Windows 允許超過 260 個字符的文件路徑。要使用此 `.reg` 檔案，你只需將上述內容儲存為 `.reg` 檔案，然後雙擊它來應用設定，或在管理員模式下的命令提示符或 PowerShell 中運行它。這是一種非常方便快速地修改系統設定的方法。
    
- [Windows Registry 註冊表(登錄檔)是什麼](https://zh.wikipedia.org/zh-tw/%E6%B3%A8%E5%86%8C%E8%A1%A8)
    
    Windows 註冊表是一個用於儲存系統設定和配置資料的資料庫，它包含了操作系統、已安裝的應用程式和硬體設備的重要資訊。註冊表中的資訊被組織在稱為「鍵」的階層結構中，每個鍵可以包含子鍵和值，並可用來控制系統和應用程式的行為。
    
    註冊表是 Windows 系統中一個非常核心的部分，因此，對註冊表的修改應該非常謹慎，錯誤的修改可能會導致系統不穩定或無法啟動。這也是為什麼有時會使用 `.reg` 檔案來精確控制註冊表修改的原因，因為它們可以提供一個確定的、可以被複查的修改過程。
    
    在 Windows 系統中，你可以透過註冊表編輯器（Regedit）來查看和修改註冊表，這是一個內建的工具，可以讓你訪問和修改註冊表中的鍵和值。
    

[什麼是 AWS Serverless Application Model (AWS SAM)？ - AWS Serverless Application Model](https://docs.aws.amazon.com/zh_tw/serverless-application-model/latest/developerguide/what-is-sam.html)

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2012.png)

## Prerequisites

Complete all [prerequisites](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/prerequisites.html) in the previous section before moving forward. This includes:

1. Signing up for an AWS account.
2. Creating an administrative IAM user.
3. Creating an access key ID and secret access key.
4. Installing the AWS CLI.
5. Configuring AWS credentials.

## 安裝 AWS SAM CLI

Mac 的用戶要注意一下，從 2023/9 開始，[AWS 不會在維護 AWS SAM CLI 的 Homebrew Installer](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)

由於我現在是使用 Windows 作業系統的電腦，今天示範如何在 Windows 安裝 AWS SAM CLI

### Windows 安裝

Windows 安裝相當簡單，只要去[官方文檔](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html#install-sam-cli-instructions)裡面下載 MSI File，接著無腦的 Next 按按按就裝好了 XD

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2013.png)

下載好之後，輸入指令 `sam --version` 檢查是否安裝成功

```bash
$ sam --version
SAM CLI, version 1.115.0
```

### Windows 啟用 **`LongPathsEnabled`**

到這邊還沒有結束，對於 Windows 用戶，Windows 系統的最大路徑限制（MAX_PATH）通常是 260 個字符，請一定要啟用 **`LongPathsEnabled` ，不然在 sam 的某些指令執行後會因為文件路徑過長出現 Error，例如: `sam init`** 

> Starting in Windows 10, version 1607, **MAX_PATH** limitations have been removed from common Win32 file and directory functions. However, you must opt-in to the new behavior.

資料來源: [https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation?tabs=powershell#enable-long-paths-in-windows-10-version-1607-and-later](https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation?tabs=powershell#enable-long-paths-in-windows-10-version-1607-and-later)
> 
> 
> [Maximum Path Length Limitation - Win32 apps](https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation?tabs=powershell#enable-long-paths-in-windows-10-version-1607-and-later)
> 

請你以系統管理員身分打開你的 Powershell，這邊我使用 Powershell 版本為 7.4.2

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2014.png)

輸入指令:

```bash
$ New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force

LongPathsEnabled : 1
PSPath           : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem
PSParentPath     : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control
PSChildName      : FileSystem
PSDrive          : HKLM
PSProvider       : Microsoft.PowerShell.Core\Registry
```

因為有些 process 可能在設置此鍵之前就已經緩存，為了讓系統上的所有應用程序識別這個鍵的值，**請重新開機！**

## Hello World Application

### 建立一個 Project

首先我們先建立一個 Project

```bash
$ sam init

        SAM CLI now collects telemetry to better understand customer needs.

        You can OPT OUT and disable telemetry collection by setting the
        environment variable SAM_CLI_TELEMETRY=0 in your shell.
        Thanks for your help!

        Learn More: https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-telemetry.html

You can preselect a particular runtime or package type when using the `sam init` experience.
Call `sam init --help` to learn more.

Which template source would you like to use?
        1 - AWS Quick Start Templates
        2 - Custom Template Location
Choice: 1

Choose an AWS Quick Start application template
        1 - Hello World Example
        2 - Data processing
        3 - Hello World Example with Powertools for AWS Lambda
        4 - Multi-step workflow
        5 - Scheduled task
        6 - Standalone function
        7 - Serverless API
        8 - Infrastructure event management
        9 - Lambda Response Streaming
        10 - Serverless Connector Hello World Example
        11 - Multi-step workflow with Connectors
        12 - GraphQLApi Hello World Example
        13 - Full Stack
        14 - Lambda EFS example
        15 - DynamoDB Example
        16 - Machine Learning
Template: 1

Use the most popular runtime and package type? (Python and zip) [y/N]: y

Would you like to enable X-Ray tracing on the function(s) in your application?  [y/N]:

Would you like to enable monitoring using CloudWatch Application Insights?
For more info, please view https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-application-insights.html [y/N]:

Would you like to set Structured Logging in JSON format on your Lambda functions?  [y/N]:

Project name [sam-app]: aws-sam-101

Cloning from https://github.com/aws/aws-sam-cli-app-templates (process may take a moment)

    -----------------------
    Generating application:
    -----------------------
    Name: aws-sam-101
    Runtime: python3.9
    Architectures: x86_64
    Dependency Manager: pip
    Application Template: hello-world
    Output Directory: .
    Configuration file: aws-sam-101\samconfig.toml

    Next steps can be found in the README file at aws-sam-101\README.md

Commands you can use next
=========================
[*] Create pipeline: cd aws-sam-101 && sam pipeline init --bootstrap
[*] Validate SAM template: cd aws-sam-101 && sam validate
[*] Test Function in the Cloud: cd aws-sam-101 && sam sync --stack-name {stack-name} --watch
```

現在我們已經成功建立好專案

我們進到裡面看一下

```bash
$ cd aws-sam-101
$ tree

│  .gitignore
│  README.md
│  samconfig.toml # 存參數
│  template.yaml # AWS 根據此檔案配置你的 Infra
│  __init__.py
│
├─events
│      event.json
│
├─hello_world
│      app.py # 你的 Lanbda Function 寫在這
│      requirements.txt
│      __init__.py
│
└─tests
    │  requirements.txt
    │  __init__.py
    │
    ├─integration
    │      test_api_gateway.py
    │      __init__.py
    │
    └─unit
            test_handler.py
            __init__.py
```

### Build

接下來我們就要來打包我們專案了，但因為我想要使用 Python 3.11，所以我先到 `template.yaml` 把 Runtime 改成 3.11

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2015.png)

接著輸入以下指令

```bash
$ sam build

Starting Build use cache
Manifest file is changed (new hash: 3298f13049d19cffaa37ca931dd4d421) or dependency folder
(.aws-sam\deps\ab6747e1-a68c-4fab-ae91-fa1c4dcd23e1) is missing for (HelloWorldFunction), downloading dependencies and
copying/building source
Building codeuri: C:\GitHub\aws-sam-101\hello_world runtime: python3.11 metadata: {} architecture: x86_64 functions:
HelloWorldFunction
 Running PythonPipBuilder:CleanUp
 Running PythonPipBuilder:ResolveDependencies
 Running PythonPipBuilder:CopySource
 Running PythonPipBuilder:CopySource

Build Succeeded

Built Artifacts  : .aws-sam\build
Built Template   : .aws-sam\build\template.yaml

Commands you can use next
=========================
[*] Validate SAM template: sam validate
[*] Invoke Function: sam local invoke
[*] Test Function in the Cloud: sam sync --stack-name {{stack-name}} --watch
[*] Deploy: sam deploy --guided
```

如指令 output 所示，你會發現你的 .aws-sam 目錄下多了 build 這個目錄

```bash
.aws-sam
├── build
│   ├── HelloWorldFunction 
│   │   ├── __init__.py
│   │   ├── app.py # Lambda Function
│   │   └── requirements.txt
│   └── template.yaml
└── build.toml
```

### Deploy

在這個部份，你需要配置你的 AWS Credentials，我已經事先創建好一個 IAM User 來暫時用，詳細如何[配置你的 AWS Credentials 請自行翻閱官方文檔](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/prerequisites.html#prerequisites-configure-credentials)，本文預設讀者已具備操作 AWS 的基本能力

輸入以下指令來部署你的 Lambda

```bash
$ sam deploy --guided

Configuring SAM deploy
======================

        Looking for config file [samconfig.toml] :  Found
        Reading default arguments  :  Success

        Setting default arguments for 'sam deploy'
        =========================================
        Stack Name [aws-sam-101]:
        AWS Region [ap-northeast-1]:
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [Y/n]: n
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]:
        #Preserves the state of previously provisioned resources when an operation fails
        Disable rollback [y/N]:
        HelloWorldFunction has no authentication. Is this okay? [y/N]: y
        Save arguments to configuration file [Y/n]:
        SAM configuration file [samconfig.toml]:
        SAM configuration environment [default]:
```

```bash
Looking for resources needed for deployment:
        Creating the required resources...
        Successfully created!

        Managed S3 bucket: aws-sam-cli-managed-default-samclisourcebucket-jptiw4noplqk
        A different default S3 bucket can be set in samconfig.toml and auto resolution of buckets turned off by setting resolve_s3=False

        Parameter "stack_name=aws-sam-101" in [default.deploy.parameters] is defined as a global parameter [default.global.parameters].
        This parameter will be only saved under [default.global.parameters] in C:\GitHub\aws-sam-101\samconfig.toml.

        Saved arguments to config file
        Running 'sam deploy' for future deployments will use the parameters saved above.
        The above parameters can be changed by modifying samconfig.toml
        Learn more about samconfig.toml syntax at
        https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-config.html

        Uploading to aws-sam-101/37624213qqc910da9321690a64a28  554765 / 554765  (100.00%)

        Deploying with following values
        ===============================
        Stack name                   : aws-sam-101
        Region                       : ap-northeast-1
        Confirm changeset            : False
        Disable rollback             : False
        Deployment s3 bucket         : aws-sam-cli-managed-default-samclisourcebucket-jptiw4noplqk
        Capabilities                 : ["CAPABILITY_IAM"]
        Parameter overrides          : {}
        Signing Profiles             : {}

Initiating deployment
=====================

        Uploading to aws-sam-101/dbd54debe319zaa19577cbf2egaj4.template  1257 / 1257  (100.00%)

Waiting for changeset to be created..

CloudFormation stack changeset
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Operation                                   LogicalResourceId                           ResourceType                                Replacement
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
+ Add                                       HelloWorldFunctionHelloWorldPermissionPro   AWS::Lambda::Permission                     N/A
                                            d
+ Add                                       HelloWorldFunctionRole                      AWS::IAM::Role                              N/A
+ Add                                       HelloWorldFunction                          AWS::Lambda::Function                       N/A
+ Add                                       ServerlessRestApiDeployment47fcad5f9d       AWS::ApiGateway::Deployment                 N/A
+ Add                                       ServerlessRestApiProdStage                  AWS::ApiGateway::Stage                      N/A
+ Add                                       ServerlessRestApi                           AWS::ApiGateway::RestApi                    N/A
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Changeset created successfully. arn:aws:cloudformation:ap-northeast-1:1234556781:changeSet/samcli-deploy17144221345/7ac52521a5-34ce-432f-bb1e-64521743e8g

2024-04-28 21:27:31 - Waiting for stack create/update to complete

CloudFormation events from stack operations (refresh every 5.0 seconds)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ResourceStatus                              ResourceType                                LogicalResourceId                           ResourceStatusReason
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
CREATE_IN_PROGRESS                          AWS::CloudFormation::Stack                  aws-sam-101                                 User Initiated
CREATE_IN_PROGRESS                          AWS::IAM::Role                              HelloWorldFunctionRole                      -
CREATE_IN_PROGRESS                          AWS::IAM::Role                              HelloWorldFunctionRole                      Resource creation Initiated
CREATE_COMPLETE                             AWS::IAM::Role                              HelloWorldFunctionRole                      -
CREATE_IN_PROGRESS                          AWS::Lambda::Function                       HelloWorldFunction                          -
CREATE_IN_PROGRESS                          AWS::Lambda::Function                       HelloWorldFunction                          Resource creation Initiated
CREATE_IN_PROGRESS                          AWS::Lambda::Function                       HelloWorldFunction                          Eventual consistency check initiated
CREATE_IN_PROGRESS                          AWS::ApiGateway::RestApi                    ServerlessRestApi                           -
CREATE_IN_PROGRESS                          AWS::ApiGateway::RestApi                    ServerlessRestApi                           Resource creation Initiated
CREATE_COMPLETE                             AWS::ApiGateway::RestApi                    ServerlessRestApi                           -
CREATE_IN_PROGRESS                          AWS::ApiGateway::Deployment                 ServerlessRestApiDeployment432d5f9d       -
CREATE_IN_PROGRESS                          AWS::Lambda::Permission                     HelloWorldFunctionHelloWorldPermissionPro   -
                                                                                        d
CREATE_COMPLETE                             AWS::Lambda::Function                       HelloWorldFunction                          -
CREATE_IN_PROGRESS                          AWS::Lambda::Permission                     HelloWorldFunctionHelloWorldPermissionPro   Resource creation Initiated
                                                                                        d
CREATE_IN_PROGRESS                          AWS::ApiGateway::Deployment                 ServerlessRestApiDeployment4715f9d       Resource creation Initiated
CREATE_COMPLETE                             AWS::Lambda::Permission                     HelloWorldFunctionHelloWorldPermissionPro   -
                                                                                        d
CREATE_COMPLETE                             AWS::ApiGateway::Deployment                 ServerlessRestApiDeployment12d5f9d       -
CREATE_IN_PROGRESS                          AWS::ApiGateway::Stage                      ServerlessRestApiProdStage                  -
CREATE_IN_PROGRESS                          AWS::ApiGateway::Stage                      ServerlessRestApiProdStage                  Resource creation Initiated
CREATE_COMPLETE                             AWS::ApiGateway::Stage                      ServerlessRestApiProdStage                  -
CREATE_COMPLETE                             AWS::CloudFormation::Stack                  aws-sam-101                                 -
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

CloudFormation outputs from deployed stack
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Outputs
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Key                 HelloWorldFunctionIamRole
Description         Implicit IAM Role created for Hello World function
Value               arn:aws:iam::1234566781:role/aws-sam-101-HelloWorldFunctionRole-WAAUK9mx1X1E

Key                 HelloWorldApi
Description         API Gateway endpoint URL for Prod stage for Hello World function
Value               https://ffj14dx1k.execute-api.ap-northeast-1.amazonaws.com/Prod/hello/

Key                 HelloWorldFunction
Description         Hello World Lambda Function ARN
Value               arn:aws:lambda:ap-northeast-1:1234567890:function:aws-sam-101-HelloWorldFunction-714agg15dlfg
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Successfully created/updated stack - aws-sam-101 in ap-northeast-1
```

當你看到 `Successfully created/updated stack - aws-sam-101 in ap-northeast-1` 就是完成囉

我們到 AWS Console 查看一下 Lambda 和 CloudFormation

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2016.png)

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2017.png)

而 `sam deploy` 這個指令背後執行的具體步驟如下:

1. **創建 Amazon S3 存儲桶並上傳 `.aws-sam` 目錄：**
    - AWS SAM CLI 首先創建一個 Amazon S3 存儲桶（如果沒有指定現有的存儲桶）。這個存儲桶用於存儲部署過程中需要的所有文件。
    - 接著，AWS SAM CLI 將你的 `.aws-sam` 目錄上傳到這個新建的 S3 存儲桶中。`.aws-sam` 目錄通常包含編譯和打包後的應用程序代碼及其依賴文件，這些是部署到 AWS 的必需資源。
2. **將 AWS SAM 模板轉換為 AWS CloudFormation 並上傳：**
    - AWS SAM 模板是一種描述你的服務器無應用架構的文件，它使用 YAML 或 JSON 格式編寫。AWS SAM CLI 會將這個模板轉換成 AWS CloudFormation 模板。CloudFormation 是 AWS 提供的一個服務，允許用戶通過編寫模板來模型化和設定整個 AWS 資源堆棧。
    - 轉換後的模板隨後被上傳到 AWS CloudFormation 服務。這個步驟是為了準備資源的配置和管理。
3. **AWS CloudFormation 佈置資源：**
    - 一旦模板上傳到 AWS CloudFormation 服務，CloudFormation 便開始根據模板中的定義來創建和配置所需的 AWS 資源。這包括設定如函數、資料庫、網絡設置等必要的組件。
    - CloudFormation 確保所有資源都按照模板中定義的依賴關係和參數設定正確部署，並管理資源的整個生命周期。

### 調用部署上去的 Lambda Function

現在我們可以來測試看看 API Endpoint，到剛剛的 Lambda，點擊 API Gateway 找到 Endpoint

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2018.png)

```bash
$ curl {YOUR_API_ENDPOINT}

{"message": "hello world"}
```

除了直接到 AWS Console 查看 API Endpoint 之外，sam 還有提供指令來查看 API Endpoints

```bash
$ sam list endpoints --output json

[
  {
    "LogicalResourceId": "HelloWorldFunction",
    "PhysicalResourceId": "aws-sam-101-HelloWorldFunction-7111ffa",
    "CloudEndpoint": "-",
    "Methods": "-"
  },
  {
    "LogicalResourceId": "ServerlessRestApi",
    "PhysicalResourceId": "123j013dx3i",
    "CloudEndpoint": [
      "https://12313d13i.execute-api.ap-northeast-1.amazonaws.com/Prod",
      "https://12313d13i.execute-api.ap-northeast-1.amazonaws.com/Stage"
    ],
    "Methods": [
      "/hello['get']"
    ]
  }
]
```

另外我們也可以使用 `sam remote invoke` 指令來調用部署 AWS 上的 Lambda

- 但請特別注意，invoke 後面所接的參數，是你在 `template.yaml` 中所定義的 Resources 名稱，也就是 HelloWorldFunction，這跟 Serverless Framework 的 `serverless invoke` 指令概念一樣
    
    > 關於上述提到的 Serverless Framework，我有寫一篇教學文章，裡面有提到 `serverless invoke` 指令 ([連結](https://shiun.me/blog/serverless-framework-101/#%E8%AA%BF%E7%94%A8-lambda-function))
    > 

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2019.png)

```bash
$ sam remote invoke HelloWorldFunction

Invoking Lambda Function HelloWorldFunction
START RequestId: 1ec7cb08-1066-4f23-b4fd-b542a97ef27b Version: $LATEST
END RequestId: 1ec7cb08-1066-4f23-b4fd-b542a97ef27b
REPORT RequestId: 1ec7cb08-1066-4f23-b4fd-b542a97ef27b  Duration: 2.30 ms       Billed Duration: 3 ms   Memory Size: 128 MB     Max Memory Used: 33 MB  Init Duration: 85.18 ms
{"statusCode": 200, "body": "{\"message\": \"hello world\"}"}
```

### 對程式碼做個小更改，並快速部署最新變更至 AWS

現在我們來稍微修改一下 Lambda 程式碼，我把 `hello world` 改成 →  `hello shiun`

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2020.png)

也許第一時間我們會想說可以用 `sam deploy` 這個指令把最新的變更部署上去，但是 AWS SAM CLI 還提供了一個指令 — `sam snyc` ，當正在開發 AWS Lambda 函數或其他 AWS 資源並且需要頻繁進行小的更改時，`sam sync` 可以讓您快速將這些更改推送到 AWS

### sam deploy vs sam sync

- **sam sync**
    - 允許開發者快速將本地更改同步到已部署的應用，特別是對於代碼和配置的小修改。這對於快速開發周期非常有用，因為它大幅減少了等待時間，開發者可以即時看到他們更改的效果
    - 只更新有變更的資源，避免了不必要的重複部署過程，這樣可以節省時間和成本，尤其是在開發階段的頻繁更新中
- **sam deploy**：
    - 進行完整的部署，包括重新打包應用、上傳到 S3，並通過 AWS CloudFormation 更新整個 Stack。這個過程通常比較耗時，對於小幅度的迭代來說可能效率不高
    - 每次部署都可能涉及重建和重啟所有資源，即使是未更改的部分也一樣，這會導致更高的時間和成本消耗

綜合以上，我們這裡修改了程式碼，比較好的做法是使用 `sam sync` 指令來部署最新變更

```bash
$ sam sync --watch

The SAM CLI will use the AWS Lambda, Amazon API Gateway, and AWS StepFunctions APIs to upload your code without
performing a CloudFormation deployment. This will cause drift in your CloudFormation stack.
**The sync command should only be used against a development stack**.

Confirm that you are synchronizing a development stack.

Enter Y to proceed with the command, or enter N to cancel:
 [Y/n]: Y
Queued infra sync. Waiting for in progress code syncs to complete...
Starting infra sync.
Manifest is not changed for (HelloWorldFunction), running incremental build
Building codeuri: C:\GitHub\aws-sam-101\hello_world runtime: python3.11 metadata: {} architecture: x86_64 functions: HelloWorldFunction
 Running PythonPipBuilder:CopySource

Build Succeeded

Successfully packaged artifacts and wrote output template to file C:\Users\Shiun\AppData\Local\Temp\tmpve9a9m2j.
Execute the following command to deploy the packaged template
sam deploy --template-file C:\Users\Shiun\AppData\Local\Temp\tmpveam2j --stack-name <YOUR STACK NAME>

        Deploying with following values
        ===============================
        Stack name                   : aws-sam-101
        Region                       : ap-northeast-1
        Disable rollback             : False
        Deployment s3 bucket         : aws-sam-cli-managed-default-samclisourcebucket-jptialqk
        Capabilities                 : ["CAPABILITY_NAMED_IAM", "CAPABILITY_AUTO_EXPAND"]
        Parameter overrides          : {}
        Signing Profiles             : null

Initiating deployment
=====================

2024-04-28 22:15:56 - Waiting for stack create/update to complete

CloudFormation events from stack operations (refresh every 0.5 seconds)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ResourceStatus                              ResourceType                                LogicalResourceId                           ResourceStatusReason
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
UPDATE_IN_PROGRESS                          AWS::CloudFormation::Stack                  aws-sam-101                                 User Initiated
UPDATE_IN_PROGRESS                          AWS::CloudFormation::Stack                  aws-sam-101                                 Transformation succeeded
CREATE_IN_PROGRESS                          AWS::CloudFormation::Stack                  AwsSamAutoDependencyLayerNestedStack        -
CREATE_IN_PROGRESS                          AWS::CloudFormation::Stack                  AwsSamAutoDependencyLayerNestedStack        Resource creation Initiated
CREATE_COMPLETE                             AWS::CloudFormation::Stack                  AwsSamAutoDependencyLayerNestedStack        -
UPDATE_IN_PROGRESS                          AWS::Lambda::Function                       HelloWorldFunction                          -
UPDATE_COMPLETE                             AWS::Lambda::Function                       HelloWorldFunction                          -
UPDATE_COMPLETE_CLEANUP_IN_PROGRESS         AWS::CloudFormation::Stack                  aws-sam-101                                 -
UPDATE_COMPLETE                             AWS::CloudFormation::Stack                  aws-sam-101                                 -
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

CloudFormation outputs from deployed stack
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Outputs
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Key                 HelloWorldFunctionIamRole
Description         Implicit IAM Role created for Hello World function
Value               arn:aws:iam::1234567890:role/aws-sam-101-HelloWorldFunctionRole-WAAUK9mx1X1E

Key                 HelloWorldApi
Description         API Gateway endpoint URL for Prod stage for Hello World function
Value               https://g13i1111.execute-api.ap-northeast-1.amazonaws.com/Prod/hello/

Key                 HelloWorldFunction
Description         Hello World Lambda Function ARN
Value               arn:aws:lambda:ap-northeast-1:1234567890:function:aws-sam-101-HelloWorldFunction-7i6w81lfg
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Stack update succeeded. Sync infra completed.

CodeTrigger not created as CodeUri or DefinitionUri is missing for ServerlessRestApi.
Infra sync completed.
```

現在我們上去 AWS Console 查看一下 Lambda 和 CloudFormation 的變更

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2021.png)

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2022.png)

Lambda 已經成功修改這部分我想沒什麼問題，那 CloudFormation 的 Stack 顯示 "NESTED” 這是什麼？由於此文章主要以 AWS SAM 入門教學為主，我這邊簡單解釋：

- 在 AWS CloudFormation 中，Nested Stack 就是在一個主要的 Stack（想像成一個大的項目列表）裡面，可以創建和管理多個小 Stack（就像是項目列表中的子列表）。這樣做的好處是，當你有很多相似的設置或配置需要重複使用時，你可以把這些配置做成一個小 Stack，然後在其他項目中引用它，這樣就不需要每次都重寫相同的配置，可以讓整個結構更清晰，也更容易管理。
- 承上，由於這個特性，我們可以重用配置，這也是為什麼 sam sync 比較適合開發快速迭代，而且部署速度較快
    
    > 詳細請見 AWS 官方文檔: [https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html)
    > 

![Untitled](20240428%20-%20%E5%AF%A6%E7%BF%92%E6%97%A5%E8%AA%8C%209419901e5d47459e95ee99810479fc01/Untitled%2023.png)

再次調用一次，看看變更

```bash
$ curl {YOUR_API_ENDPOINT}

{"message": "hello shiun"}
```

### 透過 AWS SAM CLI 刪除部署在 AWS 上的資源

CloudFormation 的特性就是可以刪除 Stack 來把當初透過這個 Stack 創建出來的資源刪乾淨，而 AWS SAM 身為 CloudFormation 的拓展，也提供了 `sam delete` 這個指令來刪除該專案部署在雲端上的資源

```bash
$ sma delete

Are you sure you want to delete the stack aws-sam-101 in the region ap-northeast-1 ? [y/N]: y
        Do you want to delete the template file 3141123123abc82d45ef55faf04.template in S3? [y/N]: y
        - Deleting S3 object with key 4c36d2ab78169da9e38fe8339b80626a
        - Deleting S3 object with key bd8e9f5130e915b480c3b2279a8baedb.template
        - Deleting S3 object with key 21410647a9e8cabc82d45e5556b6a804.template
        - Deleting Cloudformation stack aws-sam-101

Deleted successfully
```
