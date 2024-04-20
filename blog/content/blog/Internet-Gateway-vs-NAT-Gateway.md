---
title: 'AWS Internet Gateway vs NAT Gateway 及使用心得分享'
date: 2024-04-21T01:04:16+08:00
draft: false
description: ""
tags: ["Network", "NAT Gateway", "Internet Gateway", "AWS", "VPC"]
categories: ["Network", "AWS"]
keywords:
- AWS
- VPC
- Internet Gateway
- NAT Gateway
- Network
---

在準備 SAA 的過程中，我覺得最難的部分就是網路，這篇文章主要先介紹一下 IGW 和 NAT Gateway 的差異，接著介紹一下一些使用心得。
另外分享一下活動，最近我們大使推出了一個「證照陪跑計畫」，可以透過這個活動拿到 50% 折價券、AWS 贈品、考照學習資源以及加入 DC 社群，一直招募到 2024/04/28，有興趣的讀者可以來報名！([報名表單連結](https://www.surveycake.com/s/nvwem))

## Internet Gateway (IGW)

IGW 是一種允許 VPC（Virtual Private Cloud）與 Internet 之間通信的 VPC組件。它能讓 VPC 內的資源如 EC2 Instance 直接訪問 Internet，同時也能讓Internet 上的使用者訪問 VPC 內的資源。

**主要功能包括**：

- **雙向通信支持**：允許配有 Public IP 的 Instance 訪問 Internet，同時也能接收來自 Internet 的數據。
- **高度的可靠性和擴展性**：確保無需用戶干預即可維持服務的持續可用。

## NAT Gateway

NAT Gateway 是一種網路地址轉換服務，允許 Private Subnet 中的 Instance 連接到 VPC 外部的服務，同時阻止外部服務主動連接這些實例。這種設計特別適合需要訪問 Internet 但不需要從 Internet 接受直接訪問的敏感或保密環境。

**NAT Gateway 價錢:**

- NAT Gateway 收費 = NAT Gateway 開啟時間 USD 0.062 per Hour + 經過 NAT 資料處理費用 USD 0.062 per GB + (Optional) 跨 AZ 傳輸費用 0.02 per GB (發送+接收)
> 總而言之很貴，最新價錢請參考 [AWS 官方](https://aws.amazon.com/vpc/pricing/)

**主要功能包括**：

- **單向連接**：保護實例不被直接從互聯網訪問，同時允許它們安全地訪問必要的外部資源。
- **高效的地址轉換**：將出站流量從 Private IP 地址轉換為 NAT Gateway 的公共IP地址。

如果你覺得還是很難懂 NAT 是什麼，我這邊就以一個學生宿舍做個比喻吧！如果你是學生，你現在住在學生宿舍的 0413 房，你不想給別人知道你的房號是幾號。

你如果想要寄信給芬蘭的聖誕老公公，那你這時候會寫好信，收件地址是聖誕老公公的住址這無庸置疑，但是寄件人地址，你會以學校警衛室做為地址，你會透過警衛室的名義來寄出，所以當你的親戚收到信件時，他會看到寄信來的是學校警衛室寄出的；接著你的親戚要回信給你，親戚會寄到學生宿舍的警衛室收，警衛室會再把這封信轉傳給你，總而言之你寄出去的信，都會以學校警衛室的名義發出去 (至於警衛為什麼會知道進來的信要轉發給誰？關鍵字: NAT Translation table)

## 核心差異與選擇考量

- **訪問權限**：IGW 允許雙向通信，適合需要與 Internet 雙向交互的公開子網應用，而 NAT Gateway 只支持出站訪問，適用於需要保護的 Private Subnet 環境。
- **實例需求**：每個 VPC 僅需一個 IGW，而每個 AZ 可能需要一個 NAT Gateway 來保證服務的高可用性。
- **成本影響**：使用 IGW 不會產生額外費用，而 NAT Gateway 則根據創建和使用情況收取費用。

## 一些使用心得

- 如果你的資源不怕別人看或是給別人直接訪問也沒什麼差，就別用 NAT Gateway 了，因為很貴！！！
  - Public IP  0.12/天，除非 Instances 達到一個數量 ，大約是 25 台 Instances，才會跟 2 個 NAT Gateway 打平，那時再來考慮改成 NAT Gateway 可能就會划算一點
  > 最新價錢請參考 [AWS 官方](https://aws.amazon.com/vpc/pricing/)
- 真的需要每個 AZ 都開 NAT Gateway 嗎？以 Tokyo 這 Region 來說，你有需要 3 個 AZ 都開嗎? 我自己是覺得要達到高可用性，兩個就夠了，不然真的很花錢，對於一些小專案或是 PoC，只要一個 NAT 然後背後的 Instance 共用同一個 NAT Gateway 即可
  > 推薦文章: [AWS NAT Gateway 佈局和設定](https://9incloud.com/aws/aws-nat-gateway-layout)
