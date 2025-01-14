---
title: 'What Is the Purpose of Last Applied Configuration in Kubectl Apply'
date: 2025-01-14T23:14:17+08:00
draft: true
description: ""
tags: ["Kubernetes", "kubectl apply"]
categories: ["Kubernetes"]
keywords:
- 
---
在 [我的 Notion Kubernetes 學習筆記 - Jan 13, 2025](https://shiun.notion.site/kubectl-apply-Last-applied-configuration-17aea7e0d9d08032a08ef251c7b55f1e?pvs=4) 中，我透過 CKA 的課程學習到 `kubectl apply` 背後的原理，而我知道 `kubectl apply` 背後有三個東西來決定它如何變更資源：

- Local file (New Configuration)
- Last applied configuration
- Live state (Live Configuration)

而我就很納悶，為什麼會需要三個東西來比較資源該如何更新？為什麼不用 Local file 和 Live state 就好？反正 Local file 就是絕對真理。

我其實問了很多次 ChatGPT 4o ，但他解釋我都不是很滿意。

後來看到 Udemy 底下的問與答有一位解釋得很不錯

> Alegandro:
>
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

---

### TL;DR

**Last Applied Configuration** 是 `kubectl apply` 判斷變更意圖的關鍵：

- **如果值之前是你設置的（在上次配置中）但現在刪除，會刪除 Live Configuration 中的值。**
- **如果值不是你設置的（不在上次配置中），則保留 Live Configuration 中的值。**

這樣設計的目的是：

1. **尊重用戶的變更意圖**：保證新配置與 **Live Configuration** 的同步。
2. **避免誤刪其他系統或用戶生成的值**：確保 Live Configuration 的穩定性和完整性。

---

### **解釋**

這段話闡述了 **`kubectl apply` 的三方合併邏輯（Three-Way Merge）**，以及為什麼需要 **Last Applied Configuration（上次應用的配置）**。核心思想是：**尊重變更意圖，正確處理字段的更新和刪除**。

---

### **具體情境與行為**

1. **當新配置文件有某個值時**
    - **Live Configuration** 無論是舊的值還是沒有該值，`kubectl apply` 都會執行以下操作：
        - **如果值存在**：替換舊的值為新配置中的值。
        - **如果值不存在**：在 Live Configuration 中新增該值。
2. **當新配置文件中刪除了某個值**
    - 需要確認該值是否應該從 Live Configuration 中刪除，這取決於：
        - 該值是否存在於 **Last Applied Configuration**。
        - 如果存在於上次配置中，說明是有意刪除，因此需要從 Live Configuration 中刪除。
        - 如果不存在於上次配置中，說明該值是其他方式添加的，應保留。
3. **為什麼需要 Last Applied Configuration**
    - 它是判斷字段刪除意圖的唯一依據。
    - 沒有它，Kubernetes 無法區分「字段被刪除」和「字段是由其他系統動態生成」這兩種情況。

---

### Examples

**上次配置 (Last Applied Configuration):**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  key1: value1
  key2: value2

```

**Live Configuration:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  key1: value1
  key2: value2
  key3: value3

```

**New Configuration:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  key1: newValue1

```

---

### **處理邏輯**

1. **更新現有值**：
    - `key1` 的值從 `value1` 更新為 `newValue1`（因為 New Configuration 中有它）。
2. **刪除字段**：
    - `key2` 在 **New Configuration** 中被刪除，但它存在於 **Last Applied Configuration**，因此從 Live Configuration 中刪除 `key2`。
3. **保留其他來源的值**：
    - `key3` 在 **New Configuration** 和 **Last Applied Configuration** 中都不存在，但存在於 **Live Configuration**。這表明 `key3` 是通過其他方式創建的，因此保留 `key3`。

---

## Resources

- [Declarative Management of Kubernetes Objects Using Configuration Files](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
