---
title: 'Understanding Lambda Proxy Integration'
date: 2024-06-14T23:41:33+08:00
draft: true
description: ""
tags: ["AWS", "AWS API Gateway", "AWS Lambda", "Lambda Proxy Integration", "Cloud Computing", "Serverless", "Backend Development", "HTTP Request"]
categories: ["AWS", "Serverless"]
keywords:
- AWS Lambda Proxy Integration
- API Gateway Lambda integration
- Lambda function HTTP request
- Serverless architecture
- AWS API Gateway setup
- Lambda Proxy advantages
- Lambda Proxy HTTP response
- AWS cloud services
- Simplify Lambda requests
- Lambda function event object
---
# AWS API Gateway Lambda Proxy Integration 解析

在 AWS API Gateway (REST API) 創建 method 時，我們可以指定他要把請求傳到哪個 AWS 服務，而如果你選擇 Lambda，你會看到有一個選項是 Lambda Proxy Integration，那我把它勾選起來會怎樣呢？

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/426ec9bd-6a89-46b3-a15c-704b8d895a35/25c6ee31-61c3-49ab-a982-1d47dd7827dd/Untitled.png)

## 如果我把 Lambda Proxy Integration 打開

1. **完整的 HTTP 請求資訊**：啟用 Lambda Proxy Integration 後，API Gateway 會將整個 HTTP 請求（包括 HTTP 方法、資源路徑、查詢字串、標頭、負載等）打包成單一的事件物件，並傳遞給 Lambda 函數。
2. **簡化的處理流程**：Lambda 函數接收到的是一個包含了所有請求資訊的事件物件，因此不需要再通過模板或轉換就可以直接處理請求。這樣簡化了函數的輸入處理，因為 Lambda 函數已經包含了所有必要的請求資訊。
3. **控制 HTTP 響應**：Lambda 函數需要返回一個特定格式的物件，這個物件包括 HTTP 狀態碼、響應標頭和響應體。由於響應是直接由 Lambda 函數生成的，所以開發者可以完全控制響應的格式和內容。
4. **效能和成本效益**：由於請求處理的過程更加直接，減少了中間的處理和轉換步驟，可以提高處理效率，可能也會在某種程度上降低成本。

總之，選擇 Lambda Proxy Integration 可以使得與 AWS Lambda 的整合更加緊密、直接和高效。這適合需要精細控制 HTTP 響應的場景，或者希望簡化 API 與 Lambda 之間交互的架構設計。

## 來看實際的例子

我現在有一個 Lambda Function，我刻意在 return 的地方指定了一個 Header: `X-Shiun-Custom-Header" : "test"` 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/426ec9bd-6a89-46b3-a15c-704b8d895a35/74bd4b97-1ce4-4a2e-9239-66f39e384434/Untitled.png)

接著我們實際來調用 API 看看，就會發現，我在 Lambda 上面 Return 什麼，API Gateway 就是完完整整地回傳回來，可以說是我們直接在 Lambda 這邊掌控了更多東西，對於開發上會比較方便，管理起來會比較簡單，就不用一下在 Lambda 這邊要處理業務邏輯，還要特別去 API Gateway 那邊設定 Response 的 Header 要有什麼

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/426ec9bd-6a89-46b3-a15c-704b8d895a35/a88c5c10-edce-49d0-b782-518e44ad6e89/Untitled.png)

## 那我到底要不要打開 Lambda Proxy Integration?

我會高度推薦打開，因為你可以在程式碼那邊控制 Response，這我不認為是一個”簡化”，而是”降低管理複雜度”，我認為這是一個更佳的管理方式，一定會有很多人說分開管理會更具靈活性，但我覺得完全沒有，反而是在把一個原本可以很好管理的東西刻意複雜化，明明眼前直直走就可以到終點，偏偏要在路上挖一個坑

總而言之，如果你沒有使用 Proxy Integration，則需要在 API Gateway 中配置額外的轉換模板來處理輸入和輸出格式。

決定是否開啟 Lambda Proxy Integration 時，可以考慮以下幾個因素來做決策：

1. **請求處理的複雜度**：
    - 如果你的應用需要直接訪問完整的 HTTP 請求資訊，例如 HTTP 方法、標頭、查詢參數等，使用 Lambda Proxy Integration 會更方便，因為它直接將所有請求資訊作為事件物件傳遞給 Lambda 函數。
    - 如果你只需要處理請求體（Body）中的數據，或者不需要精細控制響應，則可以不使用 Lambda Proxy Integration，這樣可以通過 API Gateway 設置更具體的請求和響應轉換。
2. **響應的靈活性**：
    - Lambda Proxy Integration 允許 Lambda 函數完全控制 HTTP 響應。如果你需要根據不同的情況動態地調整 HTTP 狀態碼、標頭或者響應體，開啟它會更合適。
    - 如果 API 響應格式相對固定或者不需要在 Lambda 函數中進行大量的條件處理，可以考慮關閉此選項，讓 API Gateway 負責響應的組織。
3. **開發與維護的簡便性**：
    - 使用 Lambda Proxy Integration 可以減少在 API Gateway 中設置各種轉換規則的需求，簡化開發流程。這對於快速開發和原型製作很有幫助。
    - 如果不使用此選項，API Gateway 將需要配置更多的整合請求和整合響應模板，這會使 API 配置更加複雜，但也可能提供更高的靈活性。
4. **性能考量**：
    - 開啟 Lambda Proxy Integration 可能會輕微提高性能，因為 API Gateway 和 Lambda 之間的資料轉換較少。
    - 但如果需要進行複雜的資料轉換，保留這些轉換在 API Gateway 中進行可能會更有效，尤其是在處理大量小請求的場景下。

根據你的具體需求、應用的性質和開發團隊的偏好，選擇合適的設定。如果不確定，你可以從開啟 Lambda Proxy Integration 開始，因為它提供了最大的靈活性和簡單性，然後根據後續的開發經驗和性能表現進行調整。

## 我覺得可以把 Proxy Integration 關閉的情境

當你不需要處理整個 HTTP 請求中的所有資訊，而只想針對某些特定的參數或資料進行處理時，可以使用 API Gateway 的 Mapping Templates（映射模板）來完成。映射模板可以幫助你在數據傳遞到 Lambda 函數之前，先對這些數據進行篩選或轉換。這樣，Lambda 函數接收到的只是已經被處理過、真正需要的數據部分。

例如，假設一個 HTTP 請求中包含了用戶的名字、地址和電話號碼，但你的 Lambda 函數只需要處理電話號碼。在這種情況下，你可以設置一個映射模板來只提取電話號碼，並且將其傳遞給 Lambda 函數。這樣做可以讓你的 Lambda 函數專注於處理電話號碼相關的邏輯，而不需要在函數內部編寫額外的程式碼來忽略其他不需要的數據。

如果你啟用了 Lambda Proxy Integration，則所有請求的資訊（包括不需要的資訊）都會直接傳遞給 Lambda 函數，這可能會使得你需要在函數內部添加額外的程式碼來過濾不需要的數據。因此，在只需要處理部分請求數據的場景下，不啟用 Lambda Proxy Integration 可以讓 Lambda 函數的處理更加精簡和高效。