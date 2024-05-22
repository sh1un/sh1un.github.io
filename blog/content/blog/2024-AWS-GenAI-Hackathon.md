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
![alt text](image-1.png)

### 比賽前夕

我原本打算要用 LangGraph + LangChain 的組合來開發，但我花了三天學完 LangGraph 之後，發現自己真的沒有頭緒，用起來很力不從心，所以在比賽的前一晚我放棄使用 LangGraph，改成純用 Lambda 來構建出一個 Flow。

而我擔心黑客松 30 個小時做不完，我在比賽的前一晚目標就是先開發出一個可以動的應用，然後在黑客松兩天串隊友們的資源。

簡單來說，比賽前一晚我的目標是:
1. 串接 OpenSearch
2. 實現 RAG
3. 用 DynamoDB 儲存聊天紀錄，並確保 LLM 應用是具上下文記憶能力
4. IaC，確保黑客松當天直接一鍵部署，我是使用 AWS SAM

等最後弄完之後，大概早上 6:00 了，當時身體是累的但是睡不太著，然後就去買早餐，和隊友集合，然後去比賽啦

> 其實當天我有開 Youtube 線上直播想要把黑客松前一晚的過程記錄起來... 沒想到我 OBS 串流打開了，但是卻忘了按下「開始直播」...一直到早上和隊友集合，隊友才告訴我我的 Youtube 畫面都是「等待中」...

## 比賽 Day1 - 一整天在 502 Bad Gateway 度過

### 提案
在比賽剛開始時，評審會到各組先看我們的初步提案給予我們建議，以我們這組「智慧移動 - Gogoro」來說，評審有: Gogoro 副總, Gogoro 資訊技術總監, Gogoro 資料科學家,  AWS Sr. SA，但 Day1 提案的時候印象中是沒看到副總出席，而我們這組就是把[架構圖](https://app.eraser.io/workspace/OvsOZInft271CWwmG1hL?origin=share)展示給評審看，同時也告訴我們遇到的難點。

![image](https://github.com/sh1un/sh1un.github.io/assets/85695943/30e38a4f-1799-439e-896f-852da61ed4aa)

> 這個題目有幾個難點:
> 1. 如何讓 LLM 回應圖片並且顯示出來?
> 2. 車主手冊有很大的比例是圖片，如何把圖片 Embed?
> 3. 車主手冊全是 pdf 檔，如何萃取出 pdf 檔內的圖片
> 4. 當使用者提供的資訊不夠詳細，LLM 這邊應要求使用者提供更多資訊
>    1. 何謂資訊足夠詳細? 該如何定義「是否詳細」
>    2. LLM 要怎麼知道還需要請使用者提供哪些額外資訊?

當時我們只有請教評審們難點 4 的建議處理方式，而評審建議我的可以使用 Chain of Thought (CoT)

### 開始卡在 502 Bad Gateway...

在比賽才剛開始，我便遇到 502 Bad Gateway，明明早上 6.7 點左右都還 run 得好好的，現在 run 就會 502 Bad Gateway，
當時我遇到的 ERROR 如下:

```shell
# 2024-05-18T01:29:19.802+-8:00
[ERROR] TypeError: not all arguments converted during string formatting
Traceback (most recent call last):
File "/var/task/is_question_relevant.py", line 186, in lambda_handler
response = retrieval_chain.invoke(
# 略
```

當時我印象中我是為了使用 LangSmith 來方便追蹤我的 Chain，所以在 `template.yaml` 為 Lambda Function 配置了一些 LangChain 相關的環境變量，所以我當時對於程式碼是沒有任何改動的，完全就是在配置一些額外的環境變量，我完全不覺得設環境變量會造成錯誤。

於是我就開始退 commit，一個一個慢慢往前追溯是哪邊開始造成的，最後把 `template.yaml` 的變更都捨棄 (也就是我配置環境變量那部分)，才發現就是真的因為配置了 LangChain 的環境變量導致發生錯誤

### 以 OpenSearch Similarity Score Threshold 決定問題是否相關

在我們的架構中，有一個節點負責判斷用戶的問題是否與 Gogoro 的主題相關。如果問題不相關，我們會禮貌地拒絕回答。我們的策略是，當用戶的提問進來時，就我們會結合 Chat History 一起去做 Embedding 然後進行 Retrieval。我們設定了一個相似度分數閾值 (Similarity Score Threshold)，只有當文檔的 Similarity Score 高於這個閾值時，文檔才會被檢索出來。因此當沒有檢索到任何文檔，我們就可以判定用戶的提問是不相關的。

而我們在 [LangChain 官方文檔](https://python.langchain.com/v0.1/docs/integrations/vectorstores/tidb_vector/#using-as-a-retriever)有看到這部分有 API 可以用，寫法會長這樣:

```python
retriever = db.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"k": 3, "score_threshold": 0.8},
)
docs_retrieved = retriever.invoke(query)
for doc in docs_retrieved:
    print("-" * 80)
    print(doc.page_content)
    print("-" * 80)
```

但是很可惜我們稍早已經測試過，實際用下去就是會看到 `NotImplementedError`，簡單來說就是 LangChain 還沒有支援 OpenSearch 使用這個 `similarity_score_threshold`:

> Based on the information you've provided and the context from the LangChain repository, it seems like you're encountering a NotImplementedError when trying to use the similarity_score_threshold search type with the OpenSearch retriever in LangChain. This is likely because the similarity_score_threshold search type is not currently supported in the OpenSearch retriever in the LangChain framework, as mentioned in this issue.
> 
> **資料來源:** https://github.com/langchain-ai/langchain/issues/13007


所以在這部分，只能放棄使用 LangChain，改成用原生的 Python 庫 [`opensearch-py`](https://pypi.org/project/opensearch-py/)，而很感謝我們組員 Toby 對這部分還算熟悉，Toby 在比賽前幾日就有用過 `opensearch-py` 實踐 RAG ([Toby 的 GitHub Repo 連結](https://github.com/fdsf53451001/gogoro_hackathon/blob/main/retrieve_data.py))，所以我省了很多研究時間。

而別以為我們現在看起來很順利，我們雖然 Retrieval 這部分已經沒問題了，但接下來要再把這部分用 LangChain 的 `chain.invoke()` 又開始卡關了，這次碰到的 ERROR 是:

```shell
[ERROR] ValueError: Invalid input type <class 'langchain_core.prompts.chat.ChatPromptTemplate'>. Must be a PromptValue, str, or list of BaseMessages.
Traceback (most recent call last):
  File "/var/task/is_question_rpy", line 219, in lambda_handler
    answer = bedrock_llm.invoke(prompt)
  File "/var/task/langchalanguage_models/chat_models.py159, in invoke
    [self._convert_input(input)],
  File "/var/task/langchalanguage_models/chat_models.py142, in _convert_input
    raise ValueError(
```

但可惜到這邊，已經晚上 6:30 左右了，所以參賽者就先回家了，而我今天一整天就是「始於 502, 終於 502」，人還沒睡覺然後第一天東西還沒辦法正常 run 心態簡直快崩潰...

一回到家心裡想著一定要把這解決才能睡，結果一到家一碰到床之後睜開眼已經是 Day2 早上 6:00


## 比賽 Day2 - 可敬的對手

### 一時來的靈感

黑客組預計會在 Day2 14:40 結束比賽

Day2 我早上 6:00 醒來，梳洗一下，也不知道為什麼... 靈感很臨時來，突然覺得我想到 Day1 Bug 的解法了，馬上打開電腦試了一下，還真的解出來了。

真的平常工作也是這樣，卡了一整天的 Bug，總是會在無意間想到解法，我也算這次很幸運這個臨感來的那麼快，在 Day2 的一早就馬上解決昨天的大難關，給 Day2 做了一個美好的開始。

現在我們的系統可以說是可以動了，放下身上的重擔後，我就趕緊搭車去黑客松比賽現場與隊友會合了。


### 將資源部署到黑客松的 AWS 帳號

其實在 Day1 所有開發的東西我都部署在自己的 AWS 帳號，因為整個專案我都是用 [AWS SAM](https://docs.aws.amazon.com/zh_tw/serverless-application-model/latest/developerguide/what-is-sam.html)來建置和部署資源的，所以要把同樣的東西部署到別的 AWS 帳號相當容易。

但當然也是沒那麼順利，過程我犯了一個很傻的錯誤，就是忘記把 `samconfig.toml` 裡面的 `image_repositories` 換掉，所以我切換到黑客松的環境後，輸入 `sam deploy`，我根本沒辦法 Push 新的 Image 上去 ECR，因為我現在的 Credential 就是沒權限去 Push Image 到我自己 AWS 帳號的 ECR

我以為是黑客松的 AWS 環境給的 Credential 有問題，所以還跑去請教 AWS SA - [Scott](https://shazi.info/mr-shazi-page/)，結果 SA 一提點我就知道自己犯蠢 XD，真的很抱歉不小心打擾到 SA

發現自己犯蠢後資源當然也部署上去了，只能說 IaC 真的讚！


### Presentation

接下來我們把資源全都部署好，Toby 用 Gradio 切了一個簡單的前端介面然後我再把前端部署上去我們的專案就大功告成了！

Eason 將會負責上台簡報，而我會負責操控電腦

在這兩天的黑客松，Eason 一直在旁邊研究我們的系統以及跟我們釐清很多技術架構，然後為我們這兩天的成品做了一個很棒的簡報

比賽 14:40 結束後，我們收到了 Gogoro 指定的問題，必須在等下的 Presentation 中進行 Live Demo。而我們「大使夢之隊」是「智慧移動」第一個要上台報告的組別。

沒過多久後，就開始上台簡報了，只能說 Eason 的台風真的很穩，邏輯清晰，把我們系統的價值和架構表達得很清楚。另外當時很多大使來旁邊幫我們加油，到了 Live Demo 環節，我們第一題不知什麼原因就 Timeout 了，但網頁重整後又正常了，真的有 Live Demo 就一定要擺綠色乖乖...而後面的幾個問題，我覺得都符合我們預期，但就是有發現我們系統的回應時間其實太久了，幾乎都 20 秒上下才完成。

但整體來說，我覺得我們已經把我們想展示的東西都展示出來了。

### 可敬的對手

在智慧移動這組，有一個勁敵是「富貴怎麼先走了」，這組的架構大概跟我們 8 成相似，但是他們在前端的展現，以及 LLM 的回應速度真的快上我們許多，當時輪到他們 Presentation 時，他們表現的非常好，當下我們都相當佩服他們 LLM 回應的速度如此之快，而且前端頁面做得舒服又有做 Steaming，那個第一印象一定是超越我們。

當時我知道他們應該就是第一名了，比賽結果出爐前，對方也有來跟我們交流他們使用的技術，我們從賽前工作坊的 GameDay 就知道他們實力雄厚了，因為賽前工作坊 GameDay 他們那組也拿了第一！

而最後結果也不出所料，是由「富貴怎麼先走了」取得智慧移動組的優勝！
最後我們在比賽結束後，跟他們聊聊天認識了一下，他們那組真的很厲害，分工相當明確，隊友間彼此都很信任，真的恭喜他們！


## 結語

這場比賽身邊蠻多的人都很看好我們大使夢之隊，也好感謝大家都來現場加油和餵食，但最後沒取得優勝自己也覺得很可惜！而「富貴怎麼先走了」這組真的很厲害，所以真的就是自己實力還要再精進！看到他們的成品可以知道自己的系統還有哪些地方可以改進

而這場比賽學到的東西真的太多了，主辦方的賽前工作坊很紮實，現場 SA 給予的技術支援也很給力，也透過各組間的交流，學習到別組的技巧

最後，真的超感謝隊友們，資料前處理那一個部分隊長 Yuna 和 Richie 超級給力！Toby 本身在做 LLM 方面的研究，在整個過程中可說是 Toby 在 LLM 這邊提供了我心目中的 Best Practice！Eason 比賽全程在一旁看著我們開發，研究 Prompt 以及釐清整體的系統架構，簡報時把系統完整的呈現出來！真的每個隊友都是核心，和隊友一同在 30 個小時內打造出應用真的會很有革命情感，很希望之後還有機會能再組隊一起參與比賽

