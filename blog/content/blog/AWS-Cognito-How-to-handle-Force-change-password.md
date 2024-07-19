---
title: '如何使用 Lambda 處理 AWS Cognito User 首次登入被系統要求強制變更密碼'
date: 2024-07-19T23:04:08+08:00
draft: false
description: "在 AWS Cognito 裡，當你在 User Pool 中創建新的 user 時，系統會要求這個 user 在第一次 login 時必須更改密碼。這是一個常見的安全措施，但如果沒有適當處理，可能會讓使用者體驗變得不太順暢。讓我們來看看如何優雅地解決這個問題。"
tags: ["AWS", "AWS Cognito", "AWS Lambda", "Serverless", "Security", "Authentication"]
categories: ["AWS"]
keywords:
- AWS Cognito
- AWS Lambda
- Password Change
- First Login
- NEW_PASSWORD_REQUIRED
- User Experience
- Boto3
- Python
- Serverless
- Security
- Cognito first login password change
- Lambda function Cognito password change
- Cognito respond_to_auth_challenge
- Python Boto3 Cognito
- Cognito user experience
- Cognito password policy
- AWS Lambda function example
- Cognito first login flow
- Cognito security

---
## 情境描述

![01-AWS-Cognito-User-Pool-Force-Change-Password](https://github.com/user-attachments/assets/b959f776-a2da-4561-aff2-f17016e4cc88)

上圖顯示的是我剛在 Cognito User Pool 中新增了一個 user，可以注意到箭頭處顯示 `Force change password`。

當這個 user 嘗試首次登入時，他們會遇到這樣的情況:

```json
{
    "message": "New password required"，
    "challengeName": "NEW_PASSWORD_REQUIRED"，
    "session": "AYABeAsYJsR3yEr0iJssKPPUPEgAHQABAAdTZXJ2aWNlABBDb2duaXRvVXNlclBvb2xzAAEAB2F3cy1rbXMAS2Fybjphd3M6a21zOnVzLXdlc3QtMjowMTU3MzY3MjcxOTg6a2V5LzI5OTFhNGE5LTM5YTAtNDQ0Mi04MWU4LWRkYjY4NTllMTg2MQC4AQIBAHhPj7k9zU4nGXUQUvM0Ccwk42DS-fm3vKmH75ktTrktNQG1gnjl6HkUVUYN1J_HPow6AAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMulTha32s34j2CmQWAgEQgDtog8CDFh2e-4YyjM2kB_MXheMgmrdY_IF3aN9TImZXddMBj7djEAPPduLZnG3ddBLYQa8x3T3WPKUkvwIAAAAADAAAEAAAAAAAAAAAAAAAAADCbdmpLPo0E4QkWLlyH8ov_____wAAAAEAAAAAAAAAAAAAAAEAAAC1oMtgshmuUU4fk36WHKBzPgJEoE1MmL0PFyhR9lRcimImOIObhhxvC1fwiYylgbYx0Gu0i1cp5Le8AvrAnUGEJjZp54TMPP4N-JCT3qSrHeq_Kat_2CuECVSQqkc1qH4z9FVOTvAnos4FrDSn2W6KvFfLo8YQh2LJxM1h3GdIeyYqj7Ipfk6PZKGYmV5P741rRMNcuYBtvE8Hq9gVqMbEPG-c5MppY_q9JoG9TyQRN7rVGlZf62_WtTqST2F3-DZPoXTMTyY"，
    "challengeParameters": {
        "USER_ID_FOR_SRP": "shiun"，
        "requiredAttributes": "[]"，
        "userAttributes": "{\"email\":\"xxxxx@gmail.com\"}"
    }
}
```

Cognito 回傳了一個 `NEW_PASSWORD_REQUIRED` 的 challenge，同時給了我們一個 session token。這就是我們需要處理的 case。

## 解決方案

要解決這個問題，我們需要提供一個 change-password 的 API endpoint，讓 user 可以順利更改密碼。整個 flow 大致如下:

1. User 首次登入
2. Cognito 回傳 `NEW_PASSWORD_REQUIRED` challenge 和 session token
3. 前端帶著 session token 呼叫我們的 change-password API
4. 密碼更改成功，user 順利登入系統

以下是一個處理這種情況的 Lambda function 範例:

```python
import json
import logging
import boto3
from botocore.exceptions import ClientError

# Initialize logger
logger = logging.getLogger()
logger.setLevel(logging.INFO)

# Initialize Cognito client
client = boto3.client("cognito-idp")

def lambda_handler(event, context):
    """
    Handles the NEW_PASSWORD_REQUIRED challenge from Cognito
    """

    try:
        # Parse the request body
        body = json.loads(event["body"])

        # Respond to the new password required challenge
        response = client.respond_to_auth_challenge(
            ClientId="YOUR_CLIENT_ID",
            ChallengeName="NEW_PASSWORD_REQUIRED",
            Session=body["session"],
            ChallengeResponses={
                "USERNAME": body["username"],
                "NEW_PASSWORD": body["new_password"],
                # Add any required attributes here if necessary
            },
        )

        logger.info("Cognito respond to challenge response: %s", response)

        # Extract access token from the response
        access_token = response["AuthenticationResult"]["AccessToken"]

        # Return successful response with the access token set in cookies
        return {
            "statusCode": 200,
            "headers": {
                "Set-Cookie": f"accessToken={access_token}; Path=/; Secure; HttpOnly; SameSite=None; Domain=.aws-educate.tw"
            },
            "body": json.dumps({"message": "Password changed successfully"}),
        }
    except ClientError as e:
        # Handle Cognito client errors
        logger.error("Cognito client error: %s", e)
        return {
            "statusCode": 400,
            "body": json.dumps({"message": e.response["Error"]["Message"]}),
        }
    except json.JSONDecodeError as e:
        # Handle JSON decoding errors
        logger.error("JSON decode error: %s", e)
        return {
            "statusCode": 400,
            "body": json.dumps({"message": "Invalid JSON format in request body"}),
        }
```

這個 Lambda function 做了以下幾件事:

1. 接收包含 username、new password 和 session token 的 request
2. 使用 Cognito 的 `respond_to_auth_challenge` API 來處理 `NEW_PASSWORD_REQUIRED` challenge
3. 如果密碼更改成功，從 response 中提取 access token
4. 將 access token 設置在 cookie 中，並返回成功訊息

透過這樣的處理，我們可以讓新用戶順利完成首次登入時的密碼更改流程，提供更好的 user experience。

記得，在實際部署時，要根據你的具體需求來調整這個 function，比如錯誤處理、日誌記錄等。同時，也要確保前端能夠正確處理這個流程，在收到 `NEW_PASSWORD_REQUIRED` challenge 時，引導用戶到密碼更改的頁面。

希望這個範例能幫助讀者更好地處理 AWS Cognito 的首次登入密碼更改流程
