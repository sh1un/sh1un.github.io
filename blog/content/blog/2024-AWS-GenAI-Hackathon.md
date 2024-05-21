---
title: '2024 AWS GenAI Hackathon'
date: 2024-05-21T22:32:41+08:00
draft: false
description: ""
tags: ["AWS", "AWS Bedrock", "Competition Experience", "Hackathon", "GenAI", "LangChain", "OpenSearch", "RAG", "Gogoro"]
categories: ["AWS", "AI", "Hackathon", "LangChain"]
keywords:
- GenAI
- Bedrock
- Claude
- Claude 3 Sonnet
- Hackathon
- AWS
- LangChain
- RAG
- Gogoro
---
來記錄一下學生時期的最後一場比賽 - [GenAI Hacathon](https://www.digitimes.com.tw/seminar/Hackathon_20240518/)，透過這篇想跟大家分享比賽的準備以及比賽的過程

## 比賽簡介
這場比賽是 DIGITIMES 主辦，AWS 作為技術支援，總共邀請六個企業來命題，分為黑客組和創意交流組（簡單來說就是技術和非技術組），而黑客組在這場比賽必須使用 AWS 相關的服務和模型依照企業命題打造出 LLM Application
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/0f15df93-d73e-4076-94fa-81573b57adde)


## 比賽前

### 賽前工作坊
在比賽前，比賽主辦方有舉辦多場賽前工作坊，讓我們熟悉 AI 技術，而我只要有空都有去參加，我把我在的工作坊的學習筆記都寫在我的 Notion 日誌
- [2024/04/27 - 基礎 AI 工作坊](https://shiun.notion.site/20240427-198bb4068249447daccb651aa8ade714?pvs=4)
- [2024/04/28 - 基礎 AI 工作坊](https://shiun.notion.site/20240428-9419901e5d47459e95ee99810479fc01?pvs=4)
- [2024/05/05 - 進階 AI 工作坊](https://shiun.notion.site/20240505-a9f4c32c17de4210959ffe3a57fcfbc4?pvs=4) (這場有 GameDay)
- [2024/05/07 - Gogoro 企業數據工作坊](https://shiun.notion.site/20240507-748a1bcd059f4714b2bbb4d920333953?pvs=4)

其實 AI 這領域真的不是我的專長領域，我自己會開始開發 GenAI 應用是從加入伊雲谷後開始的，在伊雲谷執手的專案就是一個 LLM Application，加上 GenAI 話題真的是時下最火熱的話題，活在這時代，開發的日常要不碰到 AI 真的蠻難的

而賽前工作坊的主講者們真的人都很好，講解的也都很清楚，特別感謝 Roger, Ginny 在工作坊的教導，以及 [Scott](https://shazi.info/) 在 GameDay 當天賽後的支援，在賽前工作坊也和現場的人交流，認識了不少技術高手！

只能說這次的參賽體驗真的讚，賽前可以跟這些技術大神學習真的機會難得

### 比賽題目公布
在報名時是要填寫競賽主題的志願序的，我們當初投的第一個志願是「智慧應用」第二為「智慧移動 - Gogoro」，最終我們被安排在智慧移動的組別。

而我們這組的題目如下:
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/be97638f-6e7c-4021-af85-eb453ffb2b4b)

官方提供的數據就是 Gogoro 官網上的車主手冊: https://support.gogoro.com/tw/manual/collections/236738689176894/

基本上這個命題絕對逃不過 "RAG"

這個題目有幾個難點:
1. 如何讓 LLM 回應圖片並且顯示出來?
2. 車主手冊有很大的比例是圖片，如何把圖片 Embed?
3. 車主手冊全是 pdf 檔，如何萃取出 pdf 檔內的圖片
4. 當使用者提供的資訊不夠詳細，LLM 這邊應要求使用者提供更多資訊
   1. 何謂資訊足夠詳細? 該如何定義「是否詳細」
   2. LLM 要怎麼知道還需要請使用者提供哪些額外資訊?

### 作戰會議

而在 2024/05/07 Gogoro 數據工作坊一結束，我們小組就馬上來開了第一場作戰會議，一直到比賽開始前，總共開了三次會議，我們大致把整個應用的架構想出來，以及要使用的技術棧都定義出來並做好分工。

分工部分:
- **Yuna (隊長):** 資料前處理及構建 Data Pipeline
- **Richie:** 資料前處理及構建 Data Pipeline
- **Eason:** Presentaion 及 Prompt 研究
- **Toby:** 自動化將資料 Embedding ，然後自動化將向量儲存到 OpenSearch，以及利用 Gradio 構建介面
- **Shiun (我):** LangChain 開發及部署 LLM 應用

架構部分應該看得出來我在部落格這裡塞不下，因此直接附上連結 ([連結點我](https://app.eraser.io/workspace/OvsOZInft271CWwmG1hL?origin=share))，這邊所看到的架構圖並只是我們會議討論時方便討論而畫出來的，但最後成品的架構其實跟這邊不太一樣，後面會提到最終的架構
![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/40ad9a8e-d198-4a49-9a9c-f1937882c089)

> 另外想提一下，其實自己在開發 LLM 應用時，我很喜歡使用 [Prompt Flow](https://microsoft.github.io/promptflow/) 來構建整個 Flow，雖然這是開源的，但這個終究是微軟的，所以最後這場比賽沒選擇使用 Prompt Flow XD，改成使用 [LangGraph](https://langchain-ai.github.io/langgraph/)


### 比賽前夕

比賽的前一天，我