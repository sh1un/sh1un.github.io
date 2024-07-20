---
title: '2024 伊雲谷實習計畫 - 雲端工程師實習心得'
date: 2024-07-20T23:06:18+08:00
draft: false
description: "嗨大家好我是 Shiun，這篇文章分享了 2024 我在 eCloudValley 擔任雲端工程師實習生的心得和歷程。分享了在 AWS、Azure、DevOps 及 GenAI 專案中的經歷與個人成長"
"tags": ["eCloudValley", "Cloud Engineering", "Internship", "AWS", "AWS Bedrock", "Azure", "Azure OpenAI", "OpenAI", "Claude", "DevOps", "GenAI", "Docker", "Team Collaboration", "Scrum", "Agile Development"]
"categories": ["Internship", "Cloud Computing", "AWS", "Azure", "AI", "GenAI", "Scrum", "Agile Development"]
keywords:
- eCloudValley internship
- Cloud Engineer intern experience
- AWS and Azure training
- DevOps internship
- GenAI project development
- Docker usage in internships
- Cloud computing intern insights
- Professional growth in cloud engineering
- Team collaboration in cloud projects
- Learning AWS and Azure
- Agile Development
- AWS Bedrock
- Azure OpenAI
---

大家好，我是 Shiun，今天想和大家分享我在 eCloudValley MSP 部門擔任雲端工程師實習生的經歷。這段實習讓我學到了很多實用的雲端技術，同時也在工作中遇到了許多有趣的挑戰。
![01-Group photo](https://github.com/user-attachments/assets/c807fa99-33c0-4cf6-bb0e-35c6172e6c8c)

---

## 實習工作內容

![Timeline](https://github.com/user-attachments/assets/451ee722-6348-437a-bf83-fdfde56a57e4)

在 eCloudValley 的實習期間，我主要負責以下幾項工作：

- 學習使用公有雲，包括 AWS 和 Azure。
- 開發公司內部的 GenAI 應用。
- 與同期且同部門實習生共同開發專案。
- 定期與 Mentor 開會。
- 參與最終展演。
- 撰寫系統文件。

---

## 實習期間大事紀

### 專案大改方向

在 eCloudValley 實習期間，我們的專案經歷了一次重大轉變。起初，我們的任務是開發一個 AI Bot 開單助理，目的是幫助客戶在雲端遇到問題時能夠迅速獲得支援。這個專案的初步構想是：

1. 客戶向 AI 助理求助，並向 AI 助理提供必要資訊，例如： EC2 SG Inbound Rule, 健康檢查狀態... 等等
2. 如果客戶提供的資訊不夠明確，AI 助理要明確地告知客戶還需提供哪些資訊。
3. AI 助理彙總客戶問題，轉化為技術人員易於理解的語言。
4. 將資訊轉化後，開 ticket 到開單系統。

然而，在實作過程中，我們發現這個 AI 助理的效果不如預期，且後續優化存在技術瓶頸。客戶問題過於多樣化、且問題幾乎都很具情境，不是那種有標準解答或是標準 SOP 就能解決的問題，所以 AI 助理難以處理，導致準確率低，最終還是需要人工介入，反而增加了工作量。

因此，我們在 4 月底決定將專案轉變為一個「會議錄影機器人」。這個機器人會被邀請到 LINE 群組中，默默記錄每一條訊息以及圖片 (圖片透過 Claude 3 Sonnet 取得 Caption)。TAM 只需在群組中輸入特殊指令或貼圖即可控制錄影，最終在前端頁面查看記錄並由 LLM 總結，創建 Ticket 到 MSP 部門的 Ticket System。

### 獲得 Best Tech Team

專案在 4月底、5月初大改方向，而我們必須在 5 月底展演。儘管時程緊湊，但團隊合作逐漸流暢，我們在短短不到一個月內完成了應用開發。團隊的 PO (Product Owner) 帶領我們不斷練習簡報，最終在 5/31 的展演中獲得了 Best Tech Team 的獎項，這讓我們非常開心。
![eCloudValley-Best-Tech-Team-Award](https://github.com/user-attachments/assets/3844d169-11ef-4bce-8737-4b78bf84f928)

---

## 實習中的專業學習心得

### 學會使用 AWS 及 Azure

在公有雲市場中，AWS 和 Azure 占據了領先地位。在 eCloudValley，我們通過內部的知識管理系統 (eCloudTure) 學習如何使用這些平台來部署和架設服務。這個平台讓我們可以依照課程實作 Lab，不僅學會了理論知識，還學會了動手操作。

### 學習使用 GenAI 技術

在實習期間，我們需要開發一個 GenAI 應用，幫助 TAM 自動總結客戶問題並開工單給工程師解決。我學會了使用 LangChain 框架和 Azure OpenAI, [Prompt Flow](https://microsoft.github.io/promptflow/)，但最終選擇了 AWS Bedrock 的 Claude 3 Sonnet 作為模型。學習這些技術讓我在 GenAI Hackathon 中得到了實踐機會，也讓我在使用 AI 工具時更加得心應手。

### 加強了英文口說和閱讀

由於 eCloudValley 跨足多國，內部必須使用英文交流。我們每天都要進行英文開會 (Daily Scrum)，並在最終展演時使用全英文簡報。這段經歷強迫我提升英文口說能力，讓我在短短三個月內英文口說更為流暢。

---

## 關於我喜歡 eCloudValley 的點

### 文化及地理面

1. **年輕的氛圍**：MSP 部門氣氛歡樂，而且主管們都很照顧。
2. **DevOps 主管 Timmy 重視員工想法**：會依照我們的專長和想要發展的領域來安排任務，非常重視員工想法。
3. **優秀的 Mentor**：特別是 KKueen！真的很有耐心。
    > 遇到這麼棒的 Mentor 真的很有福氣
4. **豐富的社團活動**：公司有各種社團，咖啡社特別有趣。
5. **便利的地理位置**：公司離捷運站超近，從三和國中站出來就是公司，超級方便。
![Office location](https://github.com/user-attachments/assets/44b8f115-ec6b-4390-ac30-a5d2ecb3c38c)

### 食物面

1. **喝不完的飲品**：咖啡、牛奶和豆漿應有盡有。
2. **免費午餐**：中午提供免費午餐，偶爾還有西瓜，超級爽，我都吃超多西瓜。
3. **冰棒和零食**：各種零食和冰棒供應，隨時可以享用。
4. **啤酒和調酒**：雖然很少喝，但在專案尾聲放鬆一下非常不錯。

### 學習面

1. **雲端平台練習**：公司提供 AWS 和 Azure 帳號，讓實習生可以練習雲端技術和進行 PoC。
2. **公司教育系統 - eCloudTure**：提供 Lab 環境和完整的教材，讓我們充分練習 AWS 和 Azure。
3. **技術講座**：作為 AWS Partner，公司不定期會有原廠技術講座，對於雲端技術愛好者來說非常棒。

---

## 實習之反省與檢討

### 英文的重要性

在這段實習中，我深刻體會到英文的重要性。無論是學習資源還是日常工作，英文都是不可或缺的工具。掌握好英文，才能在軟體開發這條路上如魚得水。

### 紀錄自己的成長

實習期間，我們的 Mentor 要求我們寫[日誌](https://shiun.notion.site/Shiun-s-Learning-Journal-ded4cd4a3efd47b2805f2f99dee8f9a1)，紀錄每天的學習。這讓我意識到筆記不僅是為了複習，更是為了意識到自己的成長。通過不斷的紀錄和反思，我感受到成就感，進一步提升了學習動力。

> 已經養成一個習慣了 XD，直至今日每天都在寫

---

## 結語

在 eCloudValley 的實習經歷，是我職涯中的一個重要里程碑。不僅提升了技術能力，也讓我在團隊合作和問題解決上得到了寶貴的經驗。這段實習讓我更具信心，迎接未來的挑戰。

希望我的分享對大家有所幫助，也希望有更多的機會能和大家交流技術心得。感謝大家的閱讀！
