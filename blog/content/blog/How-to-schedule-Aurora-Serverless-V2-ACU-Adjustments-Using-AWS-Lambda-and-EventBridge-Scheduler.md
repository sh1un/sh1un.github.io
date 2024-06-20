---
title: '如何使用 AWS EventBridge Scheduler 及 Lambda 自動排程調整 AWS Aurora Serverless V2 ACU'
date: 2024-04-06T23:45:32+08:00
draft: false
description: "這篇文章將教學如何使用 AWS EventBridge Scheduler 及 Lambda 自動排程調整 AWS Aurora Serverless V2 ACU"
tags: ["AWS","AWS EventBridge", "AWS EventBridge Scheduler", "AWS Lambda", "AWS Aurora", "ACU", "Serverless", "Cron", "Automated Task", "Time-based Scheduling", "AWS Aurora Serverless V2"]
categories: ["AWS","Serverless", "Database"]
keywords:
- AWS
- AWS EventBridge Scheduler
- AWS EventBridge
- AWS Lambda
- Serverless
- AWS Aurora
- ACU
- Automated Task
- Technical Tutorial
---

## EventBridge 簡介

EventBridge Scheduler 是 [AWS 在 2022/11 推出的新服務](https://aws.amazon.com/blogs/compute/introducing-amazon-eventbridge-scheduler/)，相較於傳統事件驅動的 EventBridge，新推出的 Scheduler 是時間驅動的一個服務，你可以很輕易的在上面設置一些排程任務去調用 AWS 的其他服務，截至 2024/04/06，官方文件是顯示可以調用 AWS 超過 270 種服務，就我目前使用下來的心得，真的是相當易用！

常見的使用場景:

- 自動調整服務容量: 如 Amazon ECS 任務的數量或今天要介紹的 Aurora Serverless V2 ACU (今天要示範的)
- 自動化維護任務: 定時啟動或停止 EC2 Instance，以節省成本或進行系統維護。
- SasS 訂閱即將到期通知: 蠻多 SaaS 系統可能會需要在用戶快到期時發送信件通知客戶續訂

---

## 架構說明

關於本文，我會預設讀者們對於 AWS 有基本的操作能力和認識，對於一些較瑣碎的動作會省略不講解

本文要教學的是排程每天固定時間，會透過 EventBridge Scheduler 調用 Lambda Function 然後將 Aurora Serverless V2 的 ACU 降低。

關於這個動作，我們其實要拆解成兩個部分:

1. 用 Lambda 去調整 Aurora Serverless v2 ACU
1. 用 EventBridge 去 Trigger Lambda

下圖是從 [AWS 官方 Blog](https://aws.amazon.com/blogs/compute/introducing-amazon-eventbridge-scheduler/) 下載下來的架構圖，純示意圖大概讓各位認識 EventBridge 和 Lambda 的配合
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/fbf058d1-4754-4bbb-8e44-9b0f7d8ed3e5)

整體的步驟大概會是

1. 創建 IAM Role
1. 創建 Lambda Function
1. 創建 EventBridge Scheduler

---

## 創建 IAM Policy 及 IAM Role

這個 IAM Role 將會給我們的 Lambda Function 可以去讀取和修改 Aurora

- 創建 IAM Policy
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/898c62da-ce42-43fd-945d-d4c885959fa2)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "rds:DescribeDBClusters",
                "rds:ModifyDBCluster"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:ap-northeast-1:784523829721:*"
        },
        {
            "Effect": "Allow",
            "Action": "logs:CreateLogStream",
            "Resource": "arn:aws:logs:ap-northeast-1:784523829721:log-group:/aws/lambda/AdjustAuroraACU:*"
        },
        {
            "Effect": "Allow",
            "Action": "logs:PutLogEvents",
            "Resource": "arn:aws:logs:ap-northeast-1:784523829721:log-group:/aws/lambda/AdjustAuroraACU:log-stream:*"
        }
    ]
}

```

- 創建 IAM Role，然後 Attach 剛才所創建的 Policy
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/2da5ef65-acc6-40af-bf6e-3d666e0705af)

- 創建完畢應該會類似下圖
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/e320acd0-183e-4bc8-9b2f-491ef714a12f)

## 創建 Lambda Function

- Runtime: Python 3.12
- IAM Role: {你剛才創建的 IAM Role}

```python
import boto3

def lambda_handler(event, context):
    # Extract ACU configuration and cluster identifier from the event
    min_capacity = event.get('min_capacity')
    max_capacity = event.get('max_capacity')
    cluster_identifier = event.get('cluster_identifier')

    # Ensure cluster_identifier is provided
    if not cluster_identifier:
        print("Error: 'cluster_identifier' not provided in the event.")
        return {
            'statusCode': 400,
            'body': "'cluster_identifier' is required."
        }

    # Modify the capacity for Aurora Serverless v2
    modify_aurora_serverless_v2_capacity(cluster_identifier, min_capacity, max_capacity)

def modify_aurora_serverless_v2_capacity(cluster_identifier, min_capacity, max_capacity):
    # Initialize the RDS client
    rds_client = boto3.client('rds')

    try:
        response = rds_client.modify_db_cluster(
            DBClusterIdentifier=cluster_identifier,
            ServerlessV2ScalingConfiguration={
                'MinCapacity': min_capacity,
                'MaxCapacity': max_capacity
            },
            ApplyImmediately=True
        )
        print(f"Capacity update successful: {response}")
        return {
            'statusCode': 200,
            'body': f"Capacity updated to Min: {min_capacity}, Max: {max_capacity} for {cluster_identifier}"
        }
    except Exception as e:
        print(f"Error updating capacity: {e}")
        return {
            'statusCode': 500,
            'body': f"Error updating capacity for {cluster_identifier}: {e}"
        }


```

- 進到 Configuration - General Configuration 調整 Lambda Timeout，我這邊是改成 10s
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/34d41b8b-7984-4225-849b-860352e05c85)

---

## 創建 EventBridge Scheduler

1. 搜尋 EventBridge 後進入該頁面，側邊選單點選 Scheduler/Schedules
1. 點擊 "Create Schedule"
1. 選擇 "Recurring schedule"
    - Timezone: (UTC+8) Asia/Taipei
    - Schedule Type: Cron-based schedule
    - Cron Expression: `02 4 * * ? *` (這邊請依照你想觸發的特定時間去更動)
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/0310badc-651b-4245-9103-c38dc17a891e)
1. Next
1. Target Detail
    - Target API: Lambda
    - Invoke: {選擇你剛才創建的 Lambda Function}
        - Payload

          ```json
          {
            "cluster_identifier": "demoAurora",
            "min_capacity": 1,
            "max_capacity": 1.5
          }
          ```

    ![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/9fdf5bb5-101c-4360-b66d-5fa23cd264c9)
1. Next
1. Permisison: Create new role for this schedule
1. Create

照著以上的步驟做到這裡就完成，照理來說應該就沒問題了

---

## Troubleshooting

1. **InvalidParameterCombination**
    ![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/3b0b51b7-c1c9-4ad4-a272-153c9d1597d6)
    這是我剛開始踩到的坑，原因是 Aurora Serverless V2 必須指定 `ServerlessV2ScalingConfiguration`

    > You can set the capacity of an Aurora DB instance with the [ModifyDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBCluster.html) API operation. Specify the ServerlessV2ScalingConfiguration parameter.
    >
    > [AWS Docs - What is Amazon EventBridge Scheduler?](https://docs.aws.amazon.com/scheduler/latest/UserGuide/what-is-scheduler.html)

---

## 相關資料

- [AWS Docs - What is Amazon EventBridge Scheduler?](https://docs.aws.amazon.com/scheduler/latest/UserGuide/what-is-scheduler.html)
- [AWS Docs - Managing Aurora Serverless v2 DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html)
