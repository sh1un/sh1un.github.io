---
title: '開箱 Amazon Aurora Serverless v2 Auto-pause Feature (ACU 0)'
date: 2024-12-04T19:32:03+08:00
draft: false
description: "這篇文章帶你開箱體驗 2024 年 Amazon Aurora Serverless v2 的新功能 - Auto-pause。這個功能能夠讓 Aurora 在閒置一段時間後，自動停止實例 (Auto-pause)，停止期間不會收取「執行實例小時數費用」 (Aurora Capacity Units = 0)，達到所謂的 Truly Serverless!!"
tags: ["AWS RDS", "Auto-pause", "Serverless", "AWS Aurora"]
categories: ["AWS", "Database", "Serverless"]
keywords:
- Aurora Serverless v2
- Auto-pause
- Pay-as-you-go database
- Serverless database solutions
- AWS database optimization
- MySQL
- AWS RDS
---
## Overview

就在 2024/11/20 AWS 釋出了重磅消息 — 「[**Amazon Aurora Serverless v2 supports scaling to zero capacity**](https://aws.amazon.com/tw/about-aws/whats-new/2024/11/amazon-aurora-serverless-v2-scaling-zero-capacity/)」，簡單來說這個功能能夠讓 Aurora 在閒置一段時間後，自動停止實例 (Auto-pause)，停止期間不會收取「執行實例小時數費用」 (Aurora Capacity Units = `0`)，達到所謂的 Truly Serverless!! 官方把這個新功能稱為 **Auto-pause** ，在這個功能釋出以前， Aurora Capacity Units (ACU) 最低最低只能設定成 `0.5`。

> 實際 RDS 定價 其實還有其他收費，像是儲存成本、 IOPS、傳輸… 等等，詳細內容敬請參考官方文件 ([連結](https://aws.amazon.com/tw/rds/pricing/))
> 
>  Amazon Aurora vs RDS for MySQL 在 us-west-2 的定價比較:
>
> - Aurora Serverless v2 每個 ACU 一小時 **0.12** 美金
> - RDS for MySQL - db.t4g.micro 一小時 **0.016** 美金
> 
>  約等於一 ACU 可以開 7.5 小時的 db.t4g.micro，建議大家要**根據自己的實際情境去好好計算成本**喔，**並不是 Serverless = 經濟實惠**。

---

## 一、Auto-pause 介紹

### 自動暫停

- **什麼是自動暫停？**
  - 當資料庫一段時間內沒有任何連線時，Aurora Serverless v2 會自動將其暫停，並釋放所有計算資源，將容量縮減到 **0 ACUs**。
  - 在暫停期間，您只需要支付儲存費用，計算資源費用暫停。
- **暫停條件**
  - 沒有任何連線（用戶活動）在指定的閒置時間內觸發。
  - 支援的閒置時間範圍：**5 分鐘 (300 秒) 至 24 小時 (86,400 秒)**。

### 自動恢復

- **什麼是自動恢復？**
  - 當資料庫收到第一個連線請求時，系統會自動恢復，並動態分配計算資源。
  - 恢復時間約為 **15 秒**。

> 唯獨一定要特別注意！如果你的 Aurora Instance 超過 24 小時以上是停止狀態，那冷啟動會超過 30 秒以上
>
>
> 官方文件描述如下:
>
> If an Aurora Serverless v2 instance remains paused more than 24 hours, Aurora can put the instance into a deeper sleep that takes longer to resume. In that case, the resume time can be 30 seconds or longer
>
> *Reference: [https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html#auto-pause-whynot](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html#auto-pause-whynot)*

---

## 二、建立 Aurora Serverless v2

注意 Engine Version！以下版本才支援 ACU 設定成 0
> - If you're using Aurora PostgreSQL, the database engine must be running at least version 16.3, 15.7, 14.12, or 13.15.
> - If you're using Aurora MySQL, the database engine must be running version 3.08.0 or higher.

1. 配置 Engine
    - 選擇 **Aurora (MySQL Compatible)**
    - **Engine version:** 3.08.0 以上
    ![image 0 - Create RDS - Engine Options](https://github.com/user-attachments/assets/a330d811-c637-445d-8cf5-9ec6fb433ebe)

2. 配置資料庫 Credential
    - 選 **Dev/Test**
    - 設定密碼並再次輸入密碼
    ![image 1 - Create RDS - Credential](https://github.com/user-attachments/assets/a86fa90e-6105-4a77-83b3-91f020318209)

3. 配置 Cluster / Instance
    - 選擇 **Aurora Standard**
    - 選擇 **Serverless v2**
    - **Minimum capacity:** `0`
    ![image 2 - Create RDS - Instance Configuration](https://github.com/user-attachments/assets/7c0ecc73-65d8-4f87-bfaa-ec0363570687)

4. 配置 Connectivity
    - Public access 打開 (生產環境不建議打開)
    - VPC security group
        - **Create new**
        - **New VPC security group name:** `aurora-serverless-v2-sg`

        > 不建議生產環境這樣用，由於這部分並非本工作坊主要教學目標，基於簡化複雜配置，才會將 RDS 的 Public access 打開，並設定允許所有來源 IP。
        > 在生產環境中，建議要切好網段，配置好 RDS Subnet Group，將 RDS 放置於 Private Subnet，並且基於最小需求設定 NACL, Security Group。Lambda 也需要部署在 VPC 而需要連到 Internet 需要配置 NAT Gateway。

    ![image 3- Create RDS - Connectivity](https://github.com/user-attachments/assets/c5d519c9-8d50-435f-b4ff-4d40258f8a1f)

5. 其餘保持預設 > **Create database**
    ![image 4 - Create RDS - Click Create database](https://github.com/user-attachments/assets/5c5c1fb7-2479-4583-aeb8-5095ab4edcbd)

---

## 三、如何觀察是否成功 Auto-pause?

注意！！！即使執行個體進入自動暫停，RDS Console 仍會顯示為 `Available`，這是 Aurora Serverless 的特性。

### 1. 查看 RDS 主控台中的執行個體狀態

- **步驟**：
    1. 進入 **AWS Management Console**。
    2. 選擇 **Amazon RDS** > **Databases**。
    3. 在資料庫清單中找到您的 **Aurora Serverless V2** 集群或執行個體。
    4. 檢查執行個體的 **狀態（Status）**。
        - 如果顯示為 **Available** 且未顯示負載數據，可能已自動暫停。
        - 注意：在暫停期間，狀態仍會顯示 **Available**，但其行為與正常運行有所不同。

---

### 2. 查看 CloudWatch 指標

AWS CloudWatch 提供了多種監控指標，可以確認執行個體是否處於自動暫停狀態。

**關鍵指標**:

1. **`ACUUtilization`**
    - 值為 `0` 時，表示執行個體已縮容到 `0 ACUs`，即已自動暫停。
2. **`ServerlessDatabaseCapacity`**
    - 值為 `0` 時，表示目前沒有計算資源分配，也意味著已經自動暫停。
3. **`CPUUtilization`**
    - 值為 `0%` 時，執行個體沒有進行任何處理，可能處於暫停狀態。

下圖為成功 Auto-pause 的範例，注意 **`ACUUtilization` Metric 有某段時間是成功完全貼平在最底 (0 percent)**

![image 5 - Aurora Serverless v2 - CloudWatch Metrics](https://github.com/user-attachments/assets/3d512390-3a5b-47a7-9d23-7d76b087dfac)

---

### 3. 查看事件記錄

AWS 會記錄執行個體的暫停與恢復相關事件。可以到 **Cluster > Logs & events** 頁籤查看 **Recent events**

![image 6 - Aurora Serverless v2 - Recent events](https://github.com/user-attachments/assets/bde7147a-4ec9-4b65-b442-56a9b2270e84)

---

## 四、Troubleshooting - 明明沒有活動為什麼 Aurora Serverless v2 沒有 Auto-pause

### 1. 自動暫停的限制與條件

#### 哪些情況下不會自動暫停？

1. **有用戶連線活動時**
    - 當存在未關閉的用戶連線，系統不會進入暫停狀態。
2. **啟用了某些功能或集成**：
    - **邏輯複製 (PostgreSQL) 或 Binlog 複製 (MySQL)**
        - 這些功能需要保持活躍，因此不支援自動暫停。
    - **RDS Proxy (很重要！！！如果有使用 RDS Proxy 會導致 Auto-pause 無法啟用喔)**
        - RDS Proxy 保持與資料庫的持續連線，導致無法暫停。
3. **跨區集群 (Aurora Global Database)**
    - 主要集群的寫入節點無法自動暫停。
    - 次要集群的節點可能會根據優先級部分支持暫停。

---

### 2. Failover Priority 與自動暫停的關係

#### Failover Priority 的行為

- 優先級 **0 和 1** 的節點通常不會自動暫停：
  - 這些節點被視為關鍵節點，通常與寫入節點（Writer Instance）行為一致。
  - 當寫入節點恢復時，這些節點也會自動恢復。

    > 如果叢集只有單一實例(寫入器)，Failover priority 不會影響 auto-pause 行為。單一實例的自動暫停僅取決於：
    >
    > 1. 最小容量設為 0 ACU
    > 2. 無使用者連線
    > 3. 達到設定的閒置時間
    > 4. 無使用不相容功能(如複製、Proxy 等)

#### 如何配置 Failover Priority？

- 可以設置不同節點的 Failover Priority：
  - 高優先級節點（如 `priority = 0 或 1`）：保持可用，不進入暫停。
  - 低優先級節點（如 `priority = 2 或更高`）：根據負載自動暫停。
  ![image 7 - Click Modify RDS Instance](https://github.com/user-attachments/assets/1f65094c-bb5a-4967-8d1b-efc4f66f335c)
  ![image 8 - Edit Failover priority](https://github.com/user-attachments/assets/80096bc6-336c-4cd5-8fbb-32e61ad0eebc)

---

### 3. 觀察 `instance.log` 來排查原因

> Aurora writes a separate log file for Aurora Serverless v2 DB instances with auto-pause enabled. Aurora writes to the log for each 10-minute interval that the instance isn't paused. Aurora retains up to seven of these logs, rotated daily. The current log file is named `instance.log`, and older logs are named using the pattern `instance.*YYYY-MM-DD*.*N*.log`.
>
> The `instance.log` provides more granular detail about the reasons why an Aurora Serverless v2 instance might or might not be able to pause.

**日誌中常見訊息與排查方法**:

- **`[INFO] No auto-pause blockers registered since time`**
  - **意義**：在設定的自動暫停時間內，沒有阻止暫停的活動。
  - **多執行個體集群的差異**：
    - 當讀取節點的活動在暫停時間結束前結束，寫入節點仍可按預期時間暫停。
- **`[INFO] Unable to pause database due to a new database activity`**
  - **意義**：執行個體嘗試暫停，但新連線請求在暫停完成前抵達，阻止了暫停。
- **`[INFO] Auto-pause blockers registered since time: list_of_conditions`**
  - **意義**：列出阻止暫停的所有條件。
  - **解決建議**：檢查列出的條件，調整配置或使用方式。

以我這邊為例，進入 **RDS Console** > 選擇 **database** 實例 > 切換至 **Logs & events** 頁籤 > **Logs** 選取 `instance/instance.log` > View
![image 9 - List Logs - instance log](https://github.com/user-attachments/assets/d3729ee5-ea76-49c3-8cf3-6b4f5fe9d223)

![image 10 - View instance log](https://github.com/user-attachments/assets/0abacefd-4cb8-477c-b472-07170394003d)

- Logs

    ```bash
    2024-11-28T01:33:12,982 [INFO] Auto-pause blockers registered since 2024-11-27T21:18:00.611Z: database activity before auto-pause timeout, continuous backup lag
    2024-11-28T01:43:15,085 [INFO] Auto-pause blockers registered since 2024-11-28T01:33:12.981Z: database activity before auto-pause timeout, continuous backup lag, service or customer maintenance action
    2024-11-28T01:53:15,591 [INFO] Auto-pause blockers registered since 2024-11-28T01:43:15.085Z: database activity before auto-pause timeout, continuous backup lag
    2024-11-28T02:03:16,185 [INFO] Auto-pause blockers registered since 2024-11-28T01:53:15.590Z: database activity before auto-pause timeout, continuous backup lag
    2024-11-28T02:13:16,790 [INFO] Auto-pause blockers registered since 2024-11-28T02:03:16.185Z: database activity before auto-pause timeout, continuous backup lag, service or customer maintenance action
    2024-11-28T02:23:17,389 [INFO] Auto-pause blockers registered since 2024-11-28T02:13:16.790Z: database activity before auto-pause timeout, continuous backup lag
    2024-11-28T02:33:17,989 [INFO] Auto-pause blockers registered since 2024-11-28T02:23:17.389Z: database activity before auto-pause timeout, continuous backup lag, service or customer maintenance action
    2024-11-28T02:43:18,586 [INFO] Auto-pause blockers registered since 2024-11-28T02:33:17.988Z: database activity before auto-pause timeout, continuous backup lag, service or customer maintenance action
    2024-11-28T02:53:19,186 [INFO] Auto-pause blockers registered since 2024-11-28T02:43:18.586Z: database activity before auto-pause timeout, continuous backup lag
    2024-11-28T03:03:19,787 [INFO] Auto-pause blockers registered since 2024-11-28T02:53:19.186Z: database activity before auto-pause timeout, continuous backup lag, service or customer maintenance action
    2024-11-28T03:13:20,386 [INFO] Auto-pause blockers registered since 2024-11-28T03:03:19.787Z: database activity before auto-pause timeout, continuous backup lag, service or customer maintenance action
    2024-11-28T03:23:20,987 [INFO] Auto-pause blockers registered since 2024-11-28T03:13:20.386Z: database activity before auto-pause timeout, continuous backup lag, service or customer maintenance action
    ----------------------- END OF LOG ----------------------
    ```

於是我連線進去 Instance 執行以下指令:

> 可以 **Enable RDS Data API** 後，去使用 **Query Editor** 連線到資料庫唷！
>
> ![image 11 - Click Query Editor](https://github.com/user-attachments/assets/59e2ce43-43cf-4c57-89a8-2cb4802f7923)
>
> 進入 **Query Editor** 後 > 可以參考下圖配置連線到 database
>
> ![image 12 - Query Editor - Connect to database](https://github.com/user-attachments/assets/75208a56-0612-48d5-a745-1346e867710e)
>

```sql
SHOW FULL PROCESSLIST
```

```sql
KILL connection_id;  -- connection_id 從 PROCESSLIST 中獲得
```

---

## 我的看法和心得

Aurora Serverless v2 的 Auto-pause 功能對我而言非常具有吸引力，特別是因為我有幾個專案都是 Fully Serverless 的架構，而那些專案都有這些特性：「間歇性負載」且「流量較低」，因此 **Pay-as-you-go** 的計費模式成為我考量的重點。

在 Aurora Auto-pause 功能推出之前，在 AWS 上要符合 **Pay-as-you-go** 特性的資料庫選擇只有 NoSQL - DynamoDB。
> 我這篇文章對 **Pay-as-you-go** 的主觀認定，有一個重要指標是「**閒置期間不會產生費用**」

由於 DynamoDB 的特性，在實作分頁功能（Pagination）時，若要實現順暢的上一頁和下一頁切換，通常會較為繁瑣，甚至需要額外的開發成本。此外，某些需求在使用關聯式資料庫時會更加直覺，尤其當資料之間存在強烈關聯性且需要高一致性支援的情境下，關聯式資料庫的優勢則更為明顯。

Aurora Serverless v2 的 Auto-pause 滿足了 Serverless 架構對彈性與低成本的需求，同時還能提供關聯式資料庫的能力。但以現況需要冷啟動超過 15 秒，甚至有時候超過 24 hrs 以上沒 Resume 資料庫會發生超過 30 秒的冷啟動，除非使用者可以忍受很高的延遲，不然我認為現況還不足以放到 Production 環境來服務我們專案的使用者。

也許是可以用一些額外機制讓資料庫提前 Resume (例如某事件發生就去觸發資料庫連線，提前讓資料庫 Resume)，但是我認為這反而引來了額外維護的成本，目前還不夠足以吸引我去投資那樣的精力維護那個機制。

我相信 AWS 日後一定會把 Aurora 的冷啟動時間優化，我推測 2025 re:Invent 應該可以看到這邊還會有新的重大突破！

---

## 相關連結

- [Shiun - 如何使用 AWS EventBridge Scheduler 及 Lambda 自動排程調整 AWS Aurora Serverless V2 ACU](https://shiun.me/blog/how-to-schedule-aurora-serverless-v2-acu-adjustments-using-aws-lambda-and-eventbridge-scheduler/)

- [Scaling to Zero ACUs with automatic pause and resume for Aurora Serverless v2 - Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-auto-pause.html)

- [Managing Aurora Serverless v2 DB clusters - Amazon Aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2-administration.html#autopause-logging-instance-log)
