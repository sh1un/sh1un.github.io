---
title: 'Kubernetes 常見問題：kubectl apply 的 Last applied configuration 用途是什麼？'
date: 2025-01-14T23:14:17+08:00
draft: falst
description: "在 Kubernetes 中，kubectl apply 背後有三個東西來決定它如何變更資源：Configuration file、Last applied configuration、Live configuration。為什麼不直接比較 Configuration file 和 Live configuration 就好？為什麼需要 Last applied configuration？"
tags: ["Kubernetes", "kubectl apply", "CKA"]
categories: ["Kubernetes"]
keywords:
- Kubernetes
- kubectl apply
- Last applied configuration
- Three-way merge
- Configuration file
- Live configuration
- Declarative Management
- Core Concepts
- CKA
- CKAD
- Kubernetes 學習筆記
- Kubernetes 常見問題
---
在 [我的 Notion Kubernetes 學習筆記 - Jan 13, 2025](https://shiun.notion.site/CKA-Kubernetes-Core-Concepts-16fea7e0d9d0806d968dff51830ccbbe?pvs=97#179ea7e0d9d080a39194c6c2e8b895e4)  中，我透過 CKA 的課程學習到 `kubectl apply` 背後的原理，而我知道 `kubectl apply` 背後有三個東西來決定它如何變更資源：

- **Configuration file**：配置文件，代表使用者的最新意圖。
- **Last applied configuration**：上次使用 `kubectl apply` 時記錄的配置，用於追蹤變更。
- **Live configuration**：目前 Kubernetes 叢集中的實際配置狀態。

> When `kubectl apply` updates the live configuration for an object, it does so by sending a patch request to the API server. The patch defines updates scoped to specific fields of the live object configuration. The `kubectl apply` command calculates this patch request using the configuration file, the live configuration, and the `last-applied-configuration` annotation stored in the live configuration.

我在學習時就很納悶也很疑惑：為什麼不直接比較 **Configuration file** 和 **Live configuration** 就好？反正 **Configuration file** 寫什麼就是絕對真理，**Live configuration** 照著聲明的期望配置去實現就好，為何這中間還需要多一個 **Last applied configuration**？

---

### TL;DR

**Last Applied Configuration** 是 `kubectl apply` 判斷變更意圖的關鍵：

- **如果值之前是你設置的（在 Last Applied Configuration 中）但現在刪除，會刪除 Live Configuration 中的值。**
- **如果值不是你設置的（不在 Last Applied Configuration 中，可能是其他用戶設置的），則保留 Live Configuration 中的值。**

這樣設計的目的是：

1. **尊重用戶的變更意圖**：保證新配置與 **Live Configuration** 的同步。
2. **避免誤刪其他系統或用戶配置的值**：確保 Live Configuration 的穩定性和完整性。

建議可以搜尋關鍵字：Three-way merge，就能理解背後的意涵

---

### 為什麼需要三個資料來源？

> 關鍵字：Three-way merge

為了解開我對這裡的迷惑，當初我問了很多次 ChatGPT 4o ，但他給的解釋我都不是很滿意。

回到正題！其實，這與 **欄位刪除意圖的判斷 (Determining the deletion intent of a field)** 有關。

假設我們只比較 **Configuration file** 與 **Live configuration**，當某個欄位在本地配置中不存在時，Kubernetes 無法確定：

1. 該欄位是使用者從配置文件中有意刪除，應從 **Live configuration** 中清除。
2. 該欄位是透過其他方式（如手動修改或其他工具）新增至 **Live configuration**，應予以保留。

這就是為什麼 **Last applied configuration** 非常重要，它是判斷欄位刪除意圖的核心依據。

後來看到 Udemy 課程底下的問與答有一位網友 Alegandro 解釋得很不錯，以下我整理成中文，後面會附上原始引文。
Alegandro 在 Udemy 問與答中的解釋 `kubectl apply` 如何處理變更：

- **配置文件的值永遠優先於 Live Configuration**。
  - **如果值存在於配置文件**：覆蓋 Live Configuration 中的值。
  - **如果值是新增加的**：在 Live Configuration 中新增該值。
  - **如果值不存在於配置文件，但存在於 Live Configuration**：
    - 如果值在 **Last applied configuration** 中，表示用戶有意刪除該值，應從 Live Configuration 中清除。
    - 如果值不在 **Last applied configuration** 中，表示該值是其他方式新增的，應予以保留。

> Alegandro:
>
> The values of the configuration file are always going to "win" over the live configuration.
>
> If the value exists, it gets replaced.
> If the value is new, it gets created
> If the value doesn't exists on the configuration file but exists on the live configuration, there can be 2 reasons:
>
> 1. The value was removed from the configuration file. In this case, you want to remove the value from the live configuration.
> 2. The value was added to the live configuration not using the configuration file. In this case, you don't want to remove the value of the live configuration.
>
> To know which of the 2 options was the reason of the change, you need the "last configuration applied".
> This way:
>
> 1. If the value was on the last configuration applied, it means that the value was removed from the configuration file, so remove it from the live configuration.
> 2. If the value was not on the last configuration applied, it means that the live configuration was created by other means, so don't remove the value.
>
> I think that the intention of this is to respect the intention of the changes, if it's in the configuration file, respect the value, if it's not, it can be because it was deleted from the configuration or because it was created in another way. In the last case, don't remove what was created by other means.
>

---

### 總結

**為什麼需要 Last Applied Configuration，**核心思想就是**尊重變更意圖，正確處理欄位的更新和刪除：**

- 它是判斷欄位刪除意圖的依據。
- 沒有它，Kubernetes 無法區分「欄位被刪除」和「欄位是由其他系統動態生成」這兩種情況。

---

## Resources

- [[CKA] Kubernetes 學習筆記 - Core Concepts - Shiun Notion Site](https://shiun.notion.site/CKA-Kubernetes-Core-Concepts-16fea7e0d9d0806d968dff51830ccbbe?pvs=97#179ea7e0d9d080a39194c6c2e8b895e4)
- [Declarative Management of Kubernetes Objects Using Configuration Files](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
- [Why is a 3-way merge advantageous over a 2-way merge?](https://stackoverflow.com/questions/4129049/why-is-a-3-way-merge-advantageous-over-a-2-way-merge)
