---
title: '[Terraform Module] 一個低成本、高靈活性，輕鬆預熱上百個 Lambda Function 的解決方案：Tag-Based Lambda Warmer'
date: 2024-12-30T22:27:20+08:00
draft: false
description: "Tag-Based Lambda Warmer 透過簡單的 Tag 篩選機制定期預熱大量 AWS Lambda Functions，輕鬆解決冷啟動問題。這個 Terraform Module 提供了一個低成本且高靈活性的解決方案，特別適合間歇性工作負載、流量低且對成本敏感的 Serverless 中小型專案。"
tags: ["AWS", "AWS Lambda", "Serverless", "AWS EventBridge", "AWS EventBridge Scheduler", "Terraform", "Serverless"]
categories: ["AWS", "Serverless"]
keywords:
- AWS Lambda
- Serverless
- AWS EventBridge
- AWS EventBridge Scheduler
- Terraform
- Terraform Module
- Tag-Based Lambda Warmer
- Cold Start
- Warm Start
- Prewarm
- Serverless Optimization
- Serverless Cost Optimization
- Serverless Best Practices
- Serverless Architecture
---

## 前言 - 現有架構的挑戰

最近，我在一個 Serverless 專案中遇到了一個讓人頭疼的挑戰。這個專案運行在**多個 Lambda Functions** 之上，為了避免冷啟動帶來的延遲問題影響使用者體驗，需要確保 Functions 處於「熱啟動」狀態（Warm Start），如何有效管理大量的預熱 (prewarm) 操作變得非常棘手。

常見的解決方案之一，是為每個 Lambda Function 配置一個 **EventBridge Scheduler**，雖然可以達到預熱效果，但：

- **管理成本高**：需要單獨為每個 Lambda Function 配置 Scheduler，數量多難以管理。
- **靈活性不足**：難以快速響應需求變更，重新部署新的 Lambda Function 時，需要修改 Scheduler 的 Target。

基於這些挑戰，我開發了 **Tag-Based Lambda Warmer Terraform Module**，提供了一種低成本、高靈活性的解決方案。

---

## Tag-Based Lambda Warmer 介紹

![AWS Tag-based Lambda Warmer Archietecure Diagram](https://github.com/user-attachments/assets/51faed13-9501-4819-b687-c91291cc6009)

**Tag-Based Lambda Warmer** 的核心理念非常簡單：

將需要被預熱 (prewarm) 的 Lambda Function **加上指定的 Tag**（例如：`Prewarm=true`）。

當你部署 Terraform Module 後，EventBridge Schduler 會定期調用 Tag-Based Lambda Warmer，Warmer 會根據 Tag 自動篩選需要被預熱的 Lambda Functions

這套解決方案特別適合以下情境：

1. **低流量、間歇性工作負載**：例如開發環境或小型應用中的 Lambda Functions，需要偶爾處於熱啟動狀態，但不需要持續高效能。
2. **多環境管理**：通過不同的 Tag（例如 `Environment=dev` 或 `Environment=prod`），可以輕鬆在多個環境中靈活部署。
3. **成本敏感型專案**：對於中小型 Serverless 應用，這種解決方案能有效降低運維成本。

> 還是要提醒，這種透過定期調用來保持熱啟動狀態的解決方案，只能讓你保持至少有一個熱啟動狀態的 Execution Environment，今天流量突然有一個高峰，碰上冷啟動是不可避免的

我已經將 Tag-Based Lambda Warmer 寫成 Terraform Module 並發布到 [Terraform Registry](https://registry.terraform.io/modules/aws-educate-tw/tag-based-lambda-warmer/aws/latest) 囉，你只需要在你的 Terraform 配置中引用模組:

```hcl
module "lambda_warmer" {
  source  = "aws-educate-tw/tag-based-lambda-warmer/aws"
  # version = "0.2.1"

  aws_region = "ap-northeast-1"

  # Optional: Custom configuration
  environment = "prod"
  prewarm_tag_key = "Project"
  prewarm_tag_value = "MyProject"
  lambda_schedule_expression = "rate(5 minutes)"
  scheduler_max_retry_attempts = 0
  invocation_type = "Event"
}
```

### 執行流程

1. **EventBridge Scheduler** 定時觸發 Tag-Based Lambda Warmer Function。
2. **Warmer Function** 調用 **Lambda API**，獲取所有 Lambda Functions 列表。
3. 根據指定的 Tag Key 和 Value，篩選出需要預熱的 Lambda Functions。
4. 非同步觸發這些函數進行預熱。
5. 執行日誌記錄在 **CloudWatch Logs**，方便後續監控與調試。

### 優點

如果某個 Lambda Function 不再需要預熱，只需 **移除 Tag** 或修改其值；如果日後又有預熱需求，只需重新加上 Tag，即可輕鬆加入預熱名單。

這種 **Tag-Based** 的設計具有以下幾個關鍵優勢：

1. **靈活性高**：
   - 可以快速調整預熱範圍，無需重新部署基礎設施。
   - 支援客製化篩選條件，例如基於專案名稱 (`Project=MyApp`) 或環境 (`Environment=prod`) 的 Tag。
2. **低成本**：
   - 只需一個 EventBridge Scheduler 和一個 Lambda Function 的組合，即可實現多個 Lambda 的集中化管理。
   - 避免了高昂的 **Provisioned Concurrency** 成本，完全按需付費。
3. **簡單易用**：
   - 部署 Module 後，唯一需要做的就是為 Lambda Functions 添加或移除 Tag，無需其他額外配置。 _(視情況需要新增一小段程式碼)_

### 注意事項

那些需要被預熱的 Lambda Functions 視情況可能需要在請求一到達時**先檢查是否是 Prewarming 操作**，避免被預熱後就執行了業務邏輯

以下是 Python 範例程式碼，你可以將以下程式碼插入至 `lambda_handler` 函數中，用來檢查是否是 Prewarming 操作：

```python
def lambda_handler(event, context):
    # Identify if the incoming event is a prewarm request
    if event.get("action") == "PREWARM":
        logger.info("Received a prewarm request. Skipping business logic.")
        return {
            "statusCode": 200,
            "body": "Successfully warmed up"
        }
```

---

## 結語

**Tag-Based Lambda Warmer** 是一個低成本、高靈活性的 Lambda 預熱解決方案。無論是小型專案還是多環境部署，它都能幫助你有效管理和優化 Lambda Functions 的冷啟動問題。

如果你對這個模組感興趣，歡迎參考以下資源：

- **GitHub Repository**: [terraform-aws-tag-based-lambda-warmer](https://github.com/aws-educate-tw/terraform-aws-tag-based-lambda-warmer)
- **Terraform Registry**: [AWS Lambda Warmer Module](https://registry.terraform.io/modules/aws-educate-tw/tag-based-lambda-warmer/aws/latest)

有任何問題，歡迎提交 [Issue](https://github.com/aws-educate-tw/terraform-aws-tag-based-lambda-warmer/issues/new)。希望這個工具能幫助你更好地管理 Serverless 專案中的 Lambda Functions！
