---
title: 'AWS PrivateLink in Depth Tutorial NLB Dual Stack and Private DNS Name Integration'
date: 2025-01-12T23:41:42+08:00
draft: true
description: ""
tags: ["", ""]
categories: [""]
keywords:
- 
---
# 如何透過 PrivateLink 存取 NLB 服務，並同時支援 Dual-stack 與 Private DNS 名稱

Tags: AWS ELB, AWS PrivateLink, Dual-stack
Category: AWS, DNS, Networking
Created time: January 8, 2025 9:52 AM
Last edited time: January 12, 2025 11:30 PM

- [x]  Deep dive into PrivateLink — [如何透過 PrivateLink 存取 NLB 服務，並同時支援 Dual-stack 與 Private DNS 名稱](https://www.notion.so/PrivateLink-NLB-Dual-stack-Private-DNS-175ea7e0d9d080dd8724ea3d8071af9c?pvs=21)

## Q&A

- 是的，但我想知道，VPC Interface Endpoint 是去問哪台 DNS 服務？為什麼會是我自己要設定 DNS?
    
    這是一個非常好的問題！讓我們一步一步來解析 **VPC Interface Endpoint** 如何使用 **DNS**，以及為什麼需要你來配置相關的 DNS 選項。
    
    ---
    
    ### **VPC Interface Endpoint 與 DNS 的關係**
    
    1. **VPC Interface Endpoint 的作用**：
        - 它是 AWS PrivateLink 的一部分，允許你的 VPC 內部資源（例如 EC2、Lambda）透過私有網路直接訪問 AWS 服務（如 S3、DynamoDB），而不需要通過公開的 Internet。
    2. **DNS 的角色**：
        - 當你在 VPC 中訪問 AWS 服務時（例如 `s3.amazonaws.com`），該域名需要被解析為 **VPC Interface Endpoint 的私有 IP**。
        - AWS 提供了一個機制，讓你可以透過 DNS 將這些域名解析到 VPC 內部的 Endpoint IP，而不是公開的 AWS 公網 IP。
    
    ---
    
    ### **那麼，VPC Interface Endpoint 問哪台 DNS 服務？**
    
    1. **Amazon 提供的內建 DNS 服務**：
        - 當你在 AWS VPC 中創建資源時，VPC 自動附帶了一個內建的 DNS Resolver（通常在 `x.x.x.2`，VPC 的第一個可用 IP）。
        - **這台內建的 DNS Server** 負責將 AWS 特定的服務名稱解析為相應的私有 IP 地址。
    2. **DNS 的解析流程**：
        - 當你的應用程序在 VPC 中發出 DNS 查詢（例如查詢 `s3.amazonaws.com`），VPC 的內建 DNS Server 會：
            1. 檢查你是否已經配置了 Interface Endpoint。
            2. 如果有，則將域名解析為該 Endpoint 的私有 IP 地址。
            3. 如果沒有，則會解析為該 AWS 服務的公開 IP 地址。
    3. **為什麼你需要設定 DNS**：
        - AWS 提供了 **選項**，讓你控制內建 DNS 是否解析為私有 IP 地址。例如：
            - 你可以讓 DNS 始終解析公開地址（如果你想強制流量走互聯網）。
            - 或者，確保所有查詢的域名都解析到私有 Endpoint（保證流量保持在 VPC 中）。
    
    ---
    
    ### **為什麼需要手動設定 DNS Options？**
    
    1. **自定義網路需求**：
        - 不同的網路環境可能有不同的需求，比如是否支持 IPv6，或是否需要同時支持 IPv4 和 IPv6。
    2. **高可用性與協議支持**：
        - 如果你使用的是 Dualstack，你需要確保 DNS Options 能解析出支持 IPv4 和 IPv6 的地址。
    3. **AWS 提供靈活性**：
        - AWS 無法預測你的每個使用場景，因此它允許你手動設定這些選項，來滿足你應用程序的具體需求。
    
    ---
    
    ### **那為什麼不是直接用 AWS 幫你處理所有 DNS 設定？**
    
    1. **成本與效率考量**：
        - 使用 VPC Interface Endpoint 意味著你流量完全走內部網路，可能有 **帶寬成本優化** 的需求（相比走公網）。
        - 但有些用戶可能不需要這麼多私有化的功能，他們希望流量直接走公網，這樣就需要靈活的配置。
    2. **支持多協議的需求**：
        - IPv4 和 IPv6 的混合使用越來越普遍，這種環境下 DNS 設定需要根據你的實際網路拓撲靈活處理。
    
    ---
    
    ### **總結**
    
    - **VPC Interface Endpoint 問的是 AWS 內建的 VPC DNS Resolver**（`x.x.x.2`）。
    - **你為什麼要設定 DNS Options？**
        - 因為 AWS 給了你靈活選擇的權限，你可以決定：
            1. 是使用 IPv4、IPv6 還是 Dualstack。
            2. 是否允許 AWS 自動幫你處理（選擇 Service defined）。
        - 這樣做是為了滿足不同的網路需求，並優化你的私有網路流量。
    
    希望這樣解釋可以幫助你更清楚整個邏輯！ 🎉
    
- 如果我是 Service Provider，我有 NLB，我可以自行配置我喜歡的 Private DNS name 嗎？需要購買域名嗎
    
    是的，身為 **Service Provider**，如果你想要為 **VPC Endpoint Service** 配置 **Private DNS name**，你需要擁有一個公共域名。以下是原因與詳細說明：
    
    ---
    
    ### **為什麼需要公共域名？**
    
    1. **AWS 的驗證要求**：
        - AWS 要求你證明你擁有該域名的控制權，這是為了確保安全性並避免其他人濫用未授權的域名。
        - 驗證的方式是透過在該域名的 DNS 設置中添加特定的 **TXT 記錄**。
    2. **Private DNS name 的運作方式**：
        - Private DNS name 是由 AWS 自動創建的 **Private Hosted Zone** 提供解析，但它需要將你的自定義域名（如 `api.mycompany.com`）映射到 VPC Endpoint 的區域性 DNS 名稱。
        - 此域名的基礎必須是公共域名，即可以在互聯網上解析的域名，例如 `mycompany.com`。
    
    ---
    
    ### **具體需求與條件**
    
    - **需要的公共域名**：
        - 你可以使用任何你擁有的公共域名（例如，你已在 Route 53、GoDaddy 等註冊的域名）。
        - 例如，如果你的域名是 `example.com`，你可以將 Private DNS name 設定為 `api.example.com` 或任何子域名（如 `service.example.com`）。
    - **驗證方式**：
        - AWS 提供一組驗證值（`Name` 和 `Value`），例如：
            
            ```
            Name: _6e86v84tqgqubxbwii1m.example.com
            Value: vpce:l6p0ERxlTt45jevFwOCp
            
            ```
            
        - 你需要在域名的 DNS 設置中新增這些值作為 **TXT 記錄**，完成域名擁有權驗證。
    
    ---
    
    ### **是否可以不使用公共域名？**
    
    - **不可以**：
        - AWS 要求 Private DNS name 的驗證必須基於公共域名，即使這個域名實際上僅用於內部解析。
        - 如果你沒有公共域名，則無法配置 Private DNS name。
    
    ---
    
    ### **解決方法**
    
    1. **如果你已有公共域名**：
        - 使用現有的域名，添加所需的 **TXT 記錄**，完成驗證即可。
    2. **如果你沒有公共域名**：
        - 需要購買一個公共域名（例如透過 Route 53、GoDaddy、Namecheap 等）。
        - 註冊後，將該域名指向 AWS 提供的 DNS 伺服器，然後新增 AWS 要求的 TXT 記錄。
    
    ---
    
    ### **其他注意事項**
    
    1. **子域名支持**：
        - 如果你已驗證了主域名（例如 `example.com`），則可以直接使用任意子域名（如 `api.example.com` 或 `service.api.example.com`），而無需再次驗證子域名。
    2. **Domain Ownership Verification 的必要性**：
        - 無論域名是否公開使用，AWS 都需要驗證域名擁有權，這是 AWS 的安全機制。
    3. **與 Gateway Load Balancer 的限制**：
        - 如果你的服務背後是 **Gateway Load Balancer**，則無法使用 Private DNS name。
    
    ---
    
    ### **總結**
    
    - **是的，配置 Private DNS name 必須要有一個公共域名。**
    - 這是因為 AWS 要求驗證域名的擁有權，確保 Private DNS name 的安全性與唯一性。
    - 如果你還沒有公共域名，建議購買一個（例如透過 AWS Route 53），然後按照 AWS 的步驟完成驗證並配置 Private DNS name。
    
    如果你需要更多幫助，隨時提出！ 😊
    
- grep -v 是什麼
    
    `grep -v` 是 Linux 中 **grep 命令**的一個選項，用來**反向過濾**行。也就是說，它會顯示不符合給定模式（Pattern）的行。
    
    ---
    
    ### **語法**
    
    ```bash
    grep -v "PATTERN" FILENAME
    
    ```
    
    - `v`：反向過濾，顯示不匹配 PATTERN 的行。
    - `"PATTERN"`：你要過濾的關鍵字或正則表達式。
    - `FILENAME`：要搜索的檔案名。
    
    ---
    
    ### **用法示例**
    
    ### **1. 基本用法**
    
    假設有一個檔案 `example.txt`，內容如下：
    
    ```
    apple
    banana
    cherry
    apple pie
    grape
    
    ```
    
    如果你執行以下命令：
    
    ```bash
    grep -v "apple" example.txt
    
    ```
    
    輸出結果：
    
    ```
    banana
    cherry
    grape
    
    ```
    
    - 只會顯示不包含 `apple` 的行（過濾掉含有 `apple` 的行）。
    
    ---
    
    ### **2. 與管道結合**
    
    如果你想過濾某些命令的輸出，可以將其與管道 (`|`) 結合使用。
    
    例如，過濾掉 Apache 日誌中的 `ELB-HealthChecker/2.0`：
    
    ```bash
    sudo tail -f /var/log/httpd/access_log | grep -v "ELB-HealthChecker/2.0"
    
    ```
    
    - 這會實時顯示訪問日誌，但會過濾掉包含 `ELB-HealthChecker/2.0` 的行。
    
    ---
    
    ### **3. 與正則表達式結合**
    
    `grep` 支援正則表達式，你可以用更複雜的模式來過濾。
    
    例如，過濾掉以數字開頭的行：
    
    ```bash
    grep -v "^[0-9]" example.txt
    
    ```
    
    ---
    
    ### **常用選項**
    
    `grep` 有許多有用的選項，以下是幾個常見的：
    
    - `i`：忽略大小寫。
    - `E`：使用擴展正則表達式。
    - `c`：只顯示匹配行的數量。
    - `n`：顯示匹配行的行號。
    
    ---
    
    ### **總結**
    
    - `grep -v` 是用來反向過濾行的工具，非常適合用於排除不想要的數據。
    - 你可以將它與其他命令組合，實現動態過濾輸出。
    
    如果你有更多相關問題，隨時問我！ 😊
    
- 終於驗證成功了，那我好奇當我輸入以下指令，是誰去解析 DNS? 是會回到我自己的 DNS 服務解析嗎？curl -4 [demoprivatelink.shiun.me](http://demoprivatelink.shiun.me/)
    
    當你執行指令 `curl -4 demoprivatelink.shiun.me` 時，**DNS 解析的具體過程**如下：
    
    ---
    
    ### **誰解析 DNS？**
    
    1. **你的本地系統（或應用程式）會啟動 DNS 查詢**
        - `curl` 會通過系統的 DNS Resolver 問題來解析域名（即 `demoprivatelink.shiun.me`）。
        - 系統會根據 `/etc/resolv.conf` 或網路設置中的 **DNS Server 配置**，向指定的 DNS Server 發送查詢請求。
    2. **DNS 查詢的路徑**
        - **如果你的機器使用的是公共 DNS 伺服器（例如 Google DNS 8.8.8.8）**：
            - DNS 查詢會被發送到公共 DNS 伺服器進行解析。
        - **如果你的機器是位於 AWS VPC 內部**：
            - 查詢會被發送到 **AWS VPC 的內建 DNS Resolver**（位於 VPC 的 `.2` 地址，例如 `x.x.x.2`）。
    3. **AWS 的行為（重寫 DNS 名稱）**
        - 如果你的 **VPC Endpoint** 中啟用了 **Private DNS**：
            - 當 DNS Resolver 問到 AWS VPC 的內建 DNS Resolver 時，AWS 會將 `demoprivatelink.shiun.me` 解析為 **VPC Endpoint 的私有 IP**。
        - 如果請求來自 **VPC 外部**，DNS Resolver 則會查詢到域名的公開 IP（例如，你原來的公開 DNS 設定）。
    
    ---
    
    ### **結論：是否會回到你自己的 DNS？**
    
    ### **1. 你的環境決定 DNS 解析的行為**
    
    - **在 AWS VPC 內部**：
        - 如果你啟用了 **Private DNS**，那麼請求會被 **AWS 的內建 DNS Resolver** 攔截並解析為 VPC Endpoint 的私有 IP。
    - **在 VPC 外部**：
        - 請求會根據你的公共 DNS 提供商（例如 Cloudflare、GoDaddy）解析，返回公開的 IP。
    
    ### **2. DNS 重寫的邏輯**
    
    - **AWS 的私有 DNS Hosted Zone**：當 VPC 內部的服務消費者請求 `demoprivatelink.shiun.me` 時，AWS 的內建 DNS Resolver 會優先解析為 VPC Endpoint 的私有 IP。
    - **你的公共 DNS 設置**：如果查詢來自 VPC 外部，則會根據你在 DNS 提供商中設置的記錄解析域名。
    
    ---
    
    ### **如何檢查誰解析了 DNS？**
    
    1. **檢查實際 IP**
        - 在執行 `curl` 前，可以用 `dig` 或 `nslookup` 確認解析的結果：
            
            ```bash
            dig demoprivatelink.shiun.me
            
            ```
            
            - 如果返回的是 VPC Endpoint 的私有 IP，則解析由 AWS VPC 的 DNS Resolver 處理。
            - 如果返回的是你的公開 IP，則解析來自你的公共 DNS 提供商。
    2. **使用 `curl` 的 `-trace` 參數檢查實際 IP**
        
        ```bash
        curl --trace - demoprivatelink.shiun.me
        
        ```
        
        - 可以追踪請求的完整路徑，確認解析到的 IP 是哪個。
    
    ---
    
    ### **模擬不同場景的解析行為**
    
    ### **在 AWS VPC 內部**
    
    - 解析域名時，AWS 的內建 DNS Resolver 會將其解析為 VPC Endpoint 的私有 IP。
    - **行為**：解析會重寫到你的 **Interface Endpoint 的私有 IP 地址**。
    
    ### **在 AWS VPC 外部**
    
    - 如果查詢域名的機器不在 VPC 內，則查詢會發送到你的公共 DNS 提供商（如 Route 53）。
    - **行為**：解析結果會是你在公共 DNS 中設置的公開 IP。
    
    ---
    
    ### **總結**
    
    - 當你執行 `curl -4 demoprivatelink.shiun.me` 時，DNS 解析是基於你的 **本地 DNS 配置**，而實際的解析邏輯會根據以下情境變化：
        1. **VPC 內部**：由 AWS VPC 的內建 DNS Resolver 重寫為私有 IP。
        2. **VPC 外部**：由你的公共 DNS 提供商解析為公開 IP。
    
    如果需要確認 DNS 查詢的來源和解析過程，可以通過 `dig` 或 `nslookup` 進一步分析。 😊
    
- 既然是在內部 DNS 解析，不是公共的，那為什麼當初還要驗證域名所有權？又不會影響到他人
    
    這是一個非常好的問題！雖然 **Private DNS name** 的解析主要發生在 **AWS VPC 的內部 DNS** 中，但 AWS 要求驗證域名所有權，背後的原因主要與安全性和防止誤用有關。以下是具體的原因和背景解釋：
    
    ---
    
    ### **1. 防止域名名稱衝突與濫用**
    
    如果 AWS 不要求驗證域名所有權，以下風險可能出現：
    
    1. **域名名稱衝突**：
        - 假設另一個服務提供者也希望使用 `demoprivatelink.shiun.me` 作為 Private DNS name，而這個域名並不屬於他們。
        - 如果沒有驗證，AWS 無法判斷該域名的真正所有者，可能導致名稱被濫用，或者解析結果與實際擁有者的配置不一致。
    2. **防止假冒服務**：
        - 如果有人惡意使用你域名的名稱（例如 `demoprivatelink.shiun.me`），將其配置為自己的 VPC Endpoint Service，可能導致流量被錯誤地導向他們的私有服務。
        - 驗證所有權確保只有域名的合法擁有者才能配置和使用這個域名。
    
    ---
    
    ### **2. 增強服務消費者的信任**
    
    服務消費者（Service Consumer）在使用 PrivateLink 時，需要信任你提供的 Private DNS name 是合法的。例如：
    
    - 如果你宣稱 `demoprivatelink.shiun.me` 是你的服務的 Private DNS name，但實際上這個域名是屬於其他人，他們的客戶端可能會被誤導。
    - 通過域名驗證，消費者可以放心該域名屬於你，進而信任該服務是由你提供的。
    
    ---
    
    ### **3. 確保 AWS DNS 重寫的準確性**
    
    在 VPC 中，AWS 的內建 DNS Resolver 會優先解析 **Private DNS name** 到 **VPC Endpoint 的私有 IP**，這個行為需要確保以下條件成立：
    
    1. **你是該域名的合法擁有者**：
        - 如果沒有驗證，AWS 無法確定 `demoprivatelink.shiun.me` 是否屬於你。
        - 域名驗證是一種保證，讓 AWS 可以正確地將這個域名解析到你的 VPC Endpoint。
    2. **避免與公共 DNS 解析結果衝突**：
        - 如果同一域名被不同的服務提供者使用，但 AWS 不進行驗證，可能會導致 VPC 內部和外部的解析結果不一致，從而產生錯誤或不穩定的行為。
    
    ---
    
    ### **4. 合規性與 AWS 的責任管理**
    
    AWS 通過域名驗證確保他們的服務不會幫助用戶侵權或濫用他人資源：
    
    1. **防止域名劫持**：
        - 如果沒有驗證，可能有人假冒他人的域名來提供私有服務，這會帶來法律和合規風險。
    2. **提升平台的安全性與公信力**：
        - 驗證流程能保證域名的合法性，進一步增強 AWS PrivateLink 的安全性和可靠性。
    
    ---
    
    ### **5. 驗證域名帶來的額外好處**
    
    驗證域名所有權除了是安全需求外，也有實際的技術好處：
    
    1. **支持公共與私有服務的統一名稱**：
        - 透過驗證，你可以同時讓 `demoprivatelink.shiun.me` 在 **公開 DNS** 和 **AWS Private DNS** 上使用，這樣服務消費者在公網或 VPC 內部都可以用相同的 DNS 名稱訪問你的服務。
    2. **跨帳戶的信任關係**：
        - 如果你的服務消費者來自多個 AWS 帳戶，域名驗證可以增強消費者對服務的信任，特別是在跨帳戶使用 PrivateLink 時。
    
    ---
    
    ### **總結：為什麼需要驗證域名？**
    
    1. **防止名稱衝突與假冒**：
        - 確保只有域名的合法擁有者才能使用這個名稱作為 Private DNS name，避免誤導或惡意濫用。
    2. **增強信任**：
        - 確保服務消費者能信任你的服務是真實且合法的。
    3. **支持 DNS 名稱一致性**：
        - 驗證後，你可以統一管理公開與私有的域名解析，方便消費者無縫訪問服務。
    4. **符合 AWS 的合規性要求**：
        - AWS 驗證域名所有權是其服務的基本安全保證，避免潛在的法律與安全風險。
    
    ---
    
    如果你對域名驗證或 PrivateLink 的其他細節還有疑問，歡迎隨時討論！ 😊
    
- 那我問一下，Route 53 private hosted zone，是否就可以隨便亂取名字？
    
    是的，**Route 53 Private Hosted Zone** 是專為 **VPC 內部 DNS 解析** 設計的，因此它的命名規則可以更加自由，**不需要遵循公共 DNS 的約束**。這意味著你可以使用任何名稱作為 Private Hosted Zone 的域名，甚至是不符合公共 DNS 規範的名稱。
    
    不過，還是有一些考量需要注意：
    
    ---
    
    ### **1. Private Hosted Zone 的命名自由度**
    
    - **可以隨意取名字**：
        - 你可以創建類似 `my.internal.zone` 或 `local.test` 的 Private Hosted Zone，甚至是一些不符合標準的名稱（例如 `mycustomname` 沒有 `.com` 或其他 TLD）。
        - 因為 Private Hosted Zone 的解析僅限於與它關聯的 VPC，所以這些名稱不會影響公共互聯網。
    - **可以與公共域名相同**：
        - 例如，如果你擁有 `example.com` 的公共域名，你也可以創建一個 Private Hosted Zone 叫 `example.com`，用來覆蓋 VPC 內部對該域名的解析。
    
    ---
    
    ### **2. 私有域名不需要域名所有權驗證**
    
    - **不需要驗證域名所有權**：
        - 與公共 DNS 不同，Private Hosted Zone 是私有的，AWS 不會檢查你是否擁有該域名的公共所有權。
        - 例如，你可以創建一個叫 `google.com` 的 Private Hosted Zone，這不會影響真正的 Google，因為它僅限於 VPC 內部解析。
    - **需要注意重名風險**：
        - 如果你的 Private Hosted Zone 名稱與公共域名重名，VPC 內部的資源將優先解析 Private Hosted Zone 的記錄。
        - 例如，VPC 內部查詢 `google.com` 時，將解析到你在 Private Hosted Zone 中配置的記錄，而不是 Google 的真實公開記錄。
    
    ---
    
    ### **3. Private Hosted Zone 的實際應用場景**
    
    ### **(1) 用於內部服務命名**
    
    - 你可以為內部服務創建專屬的域名，比如：
        - `service.internal`
        - `myapp.local`
    - 這些名稱不需要在公共 DNS 中存在。
    
    ### **(2) 覆蓋公共域名解析**
    
    - 如果你擁有 `example.com` 的公共域名，但希望在 VPC 內部有不同的解析結果，可以創建一個 Private Hosted Zone，然後定義內部解析記錄。
    
    ### **(3) 測試環境**
    
    - 測試時可以隨意創建域名，比如 `test.local` 或 `dev.myapp`，方便管理測試資源。
    
    ---
    
    ### **4. 命名的注意事項**
    
    ### **(1) 避免使用真實公共域名作測試**
    
    - 雖然你可以使用任意名稱，但如果選擇了真實的公共域名（如 `amazon.com`），容易造成混淆。
    - **建議**：對於測試用途，使用非真實的域名，例如 `.internal`、`.test` 或 `.local`。
    
    ### **(2) 與公共域名的解析優先級**
    
    - 如果 VPC 內部既有 Private Hosted Zone，又需要訪問公共網路的同名域名，解析優先順序如下：
        1. **Private Hosted Zone** 記錄優先。
        2. 如果 Private Hosted Zone 中沒有該記錄，才會查詢公共 DNS。
    
    ### **(3) 避免過多的 Private Hosted Zone 重名**
    
    - 如果多個 Private Hosted Zone 使用相同的域名，並關聯到同一個 VPC，可能會導致解析混亂。
    - **建議**：保持每個 Private Hosted Zone 的命名唯一。
    
    ---
    
    ### **5. 與公共 DNS 的比較**
    
    | **特性** | **Public DNS** | **Private Hosted Zone** |
    | --- | --- | --- |
    | **命名自由度** | 必須符合公共 DNS 規範 | 可以隨意命名，不需公共所有權驗證 |
    | **所有權驗證** | 需要驗證域名的公共所有權 | 不需要驗證 |
    | **影響範圍** | 公共互聯網 | 只限於關聯的 VPC |
    | **重名影響** | 可能與全球唯一性衝突 | 僅影響 VPC 內部解析 |
    
    ---
    
    ### **結論：可以隨便取名字，但需謹慎**
    
    1. **可以隨便命名**：Private Hosted Zone 的名稱不需要公共 DNS 規範，且僅在 VPC 內部生效，名稱可以是任何格式。
    2. **需要注意的事項**：
        - 避免與公共域名產生不必要的衝突。
        - 對於測試用途，建議使用 `.internal`、`.test` 或 `.local` 等專屬後綴。
    3. **應用範例**：
        - `myapp.local` 用於內部服務。
        - `dev.mycompany.internal` 用於測試或開發環境。
    
    如有其他具體場景或問題，歡迎討論！ 😊
    
- 所以我身為 Service Provider ，在 VPC 創建了 Endpoint Service，並沒有額外費用？
    
    根據 AWS 官方資訊，作為服務提供者（Service Provider），在 VPC 中創建 **VPC Endpoint Service** 本身並不會產生直接費用。然而，支撐該服務的基礎設施（例如 **Network Load Balancer (NLB)**）以及相關的數據傳輸可能會產生費用。
    
    以下是詳細說明：
    
    ### 1. **VPC Endpoint Service 的創建費用**
    
    - **免費**：在 VPC 中創建 **Endpoint Service** 本身是免費的，AWS 不會為此直接收取費用。
    
    ### 2. **支撐基礎設施的費用**
    
    - **Network Load Balancer (NLB)**：
        - **每小時費用**：NLB 的使用會產生每小時費用，具體費用因區域而異。
        - **數據處理費用**：通過 NLB 的數據傳輸可能會產生費用，視數據量和區域而定。
    - **公有 IPv4 地址**：
        - **使用中的公有 IPv4 地址**：與您的 AWS 資源（如 EC2 實例、負載均衡器等）關聯並正在使用的公有 IPv4 地址，每小時收費 $0.005 美元。
        - **閒置的公有 IPv4 地址**：分配給您的 AWS 帳戶但未與任何資源關聯的公有 IPv4 地址，每小時同樣收費 $0.005 美元。
    
    ### 3. **數據傳輸費用**
    
    - **跨區域流量**：如果您的服務涉及跨區域的數據傳輸，可能會產生額外的數據傳輸費用。
    
    ### 4. **總結**
    
    - **直接費用**：創建 **VPC Endpoint Service** 本身不會產生直接費用。
    - **間接費用**：支撐該服務的基礎設施（如 NLB）和數據傳輸可能會產生費用。
    
    建議您參考 [Amazon VPC 定價頁面](https://aws.amazon.com/tw/vpc/pricing/) 以獲取更詳細和最新的定價資訊。
    
    如果您有其他疑問或需要進一步的資訊，歡迎隨時提問！
    
- AWS PrivateLink 可以跨 Region 嗎？例如 Consumer 的應用都在 ap-northeast-1，但是 Service Provider 把他的 NLB 和 Endpoint Service 部署在 us-east-1
    
    抱歉，之前的回答有誤。根據最新資訊，AWS PrivateLink 現已支援跨區域連線。這表示您可以在一個區域的 VPC 中，透過介面 VPC 端點（Interface VPC Endpoint）連接到另一個區域的 VPC 端點服務（Endpoint Service）。
    
    因此，在您的情境中，位於 ap-northeast-1（東京）的 Consumer 應用程式可以透過 PrivateLink，連接到位於 us-east-1（美國東部）的 Service Provider 的 NLB 和 Endpoint Service。
    
    請注意，跨區域連線可能涉及額外的費用，建議您參考 AWS 官方文件以獲取最新的定價資訊。
    
    此外，跨區域連線的可用性可能因區域而異，建議您確認所涉及的區域是否支援此功能。
    
    [AWS](https://aws.amazon.com/tw/about-aws/whats-new/2024/11/aws-privatelink-across-region-connectivity/?utm_source=chatgpt.com)
    

- 一開始，AWS VPC Endpoint（也稱為 PrivateLink）的主要設計目的是讓 AWS VPC 內的資源，可以透過 Private IP 直接存取某些 AWS 服務，而不需要經由公網（Internet）。
你能多舉例一些實際場景嗎？例如哪個服務，在 PrivateLink 出現前，我們都只能用 Internet 去存取到那些 AWS 服務，具體狀況是怎樣？畢竟我以前沒有經歷過 PrivateLink 出現以前所遭遇的問題
    
    在 AWS PrivateLink 出現以前，對於一些 AWS 服務，VPC 內的資源（例如 EC2、Lambda 等）無法直接透過 Private IP 存取這些服務。以下是一些實際的場景，幫助你理解在 PrivateLink 出現前的具體狀況及其挑戰：
    
    1. 存取 S3 或 DynamoDB
    •	狀況：
    在 PrivateLink 出現以前，如果你的 EC2 位於 VPC 裡，想存取 S3 或 DynamoDB，必須通過公共網路，甚至需要為此配置 NAT Gateway 或 NAT Instance。
    •	流量流程：
    EC2（Private Subnet） -> NAT Gateway（或 NAT Instance） -> Internet Gateway -> AWS 公網 Endpoint（S3/DynamoDB）
    •	雖然 S3 和 DynamoDB 本身支援 SSL 加密，但這種架構中的流量依然會穿越公共網路，並依賴 Internet 路由，可能帶來：
    •	安全性風險：資料可能暴露於公網。
    •	延遲問題：流量需繞路到公網再回到 AWS 內部。
    •	解決方案後的情況：
    現在，你可以透過 PrivateLink 或 Gateway Endpoint（S3/DynamoDB 支援 Gateway Endpoint）直接建立私有路徑，完全避免公網。
    2. 存取 Kinesis 或 Systems Manager
    •	狀況：
    假設你的應用程式需要將數據流（data stream）傳送到 Amazon Kinesis，或需要使用 AWS Systems Manager（例如 Parameter Store 或 Session Manager），以前都需要透過 Internet 存取：
    •	流量流程：
    EC2（Private Subnet） -> NAT Gateway -> Internet Gateway -> Kinesis 或 Systems Manager API
    •	問題：
    •	額外成本：NAT Gateway 流量費用（Outbound 流量到 Internet）。
    •	依賴 Internet：若 Internet 出現問題或受到攻擊，可能影響內部應用與 Kinesis 的互動。
    •	解決方案後的情況：
    使用 PrivateLink，VPC 內的資源可以透過 Private IP 直接與這些服務交互，且流量始終留在 AWS Backbone 網路中。
    3. 使用 API Gateway 或 Elastic Load Balancer (ELB)
    •	狀況：
    假設你的應用程式需要存取某些 API Gateway 的 REST API 或 ALB/ELB 的應用程式後端服務：
    •	流量流程：
    EC2（Private Subnet） -> NAT Gateway -> Internet Gateway -> API Gateway 或 ELB Public Endpoint
    •	問題：
    •	安全性挑戰：即使 API Gateway 支援 SSL 加密，但流量依然經過 Internet。
    •	額外的配置需求：例如防火牆規則、IP 過濾等，來限制哪些來源 IP 能存取。
    •	解決方案後的情況：
    API Gateway 和 ALB 現在可以配置 PrivateLink，讓流量完全留在 VPC 內，提升安全性。
    4. SaaS 應用程式的存取
    •	狀況：
    假設你使用的是某些托管在 AWS 上的第三方 SaaS 服務（例如 Splunk、Snowflake 等），在 PrivateLink 出現以前，這些 SaaS 服務通常只提供 Public Endpoint。
    •	流量流程：
    EC2（Private Subnet） -> NAT Gateway -> Internet Gateway -> SaaS Public Endpoint
    •	問題：
    •	安全性風險：企業數據需暴露於公網中，可能成為攻擊目標。
    •	合規性挑戰：某些金融或醫療應用可能違反合規要求，因為敏感數據流量經過公網。
    •	解決方案後的情況：
    SaaS 提供商可通過 PrivateLink 將服務 Endpoint 暴露為 Private IP，你的 VPC 可直接存取，避免資料流量經過 Internet。
    
    總結
    
    在 PrivateLink 出現以前，依賴 NAT Gateway 進行公共網路存取是常態，但這帶來了安全性、成本和性能上的挑戰。PrivateLink 的推出，讓 VPC 資源能夠直接透過 Private IP 存取 AWS 和第三方服務，大幅提升了安全性和效率，同時減少了對 NAT Gateway 的依賴及其相關成本。
    
- 以 S3 為例，在 PrivateLink 出現前，為什麼我沒辦法走 AWS Backbone 網路？我們不是有 AWS SDK? Get object 難道透過 SDK 其實也是需要透過 internet?
    
    這是一個很好的問題！雖然我們在 AWS 裡面運行程式碼（例如透過 EC2 使用 AWS SDK 來操作 S3），直觀上會認為這應該是經過 AWS 的內部網路，但實際上，在 AWS PrivateLink 或 VPC Gateway Endpoint 出現之前，使用 AWS SDK 存取 S3 的流量的確會經過 公網（Internet）。以下是為什麼會有這種情況的原因：
    
    為什麼透過 AWS SDK 操作 S3 須經公網？
    1.	S3 的 Public Endpoint 設計
    •	在早期，Amazon S3 的存取是基於 Public Endpoint 提供的，比如：
    •	https://<bucket-name>.s3.amazonaws.com
    •	https://s3.<region>.amazonaws.com
    •	這些 Public Endpoint 是透過 公共 DNS 名稱（如 [s3.amazonaws.com](http://s3.amazonaws.com/) 或 s3.<region>.amazonaws.com）解析的，而這些 DNS 名稱的 IP 地址指向的是 AWS 的公網 IP。
    2.	AWS SDK 與 S3 通信的默認行為
    •	使用 AWS SDK（例如 boto3 或 AWS CLI）與 S3 互動時：
    •	請求會發送到這些公共 DNS 名稱。
    •	即使程式碼是在 AWS VPC 裡執行，DNS 解析的結果還是指向公網 IP。
    •	請求經由 Internet Gateway 或 NAT Gateway 到達 S3 的 Public Endpoint。
    3.	AWS S3 的架構與安全性
    •	雖然流量經過公網，但 S3 的傳輸層安全性（例如 HTTPS 加密）確保了數據的安全性。這是早期 AWS 提供的主要安全措施。
    •	但由於流量經過公網，依然可能遇到：
    •	額外的延遲：因為流量繞道到公網再回到 AWS 內部。
    •	合規性問題：某些合規性要求（如金融或醫療行業）禁止數據流量經過公網。
    
    為什麼無法直接走 AWS 內部網路？
    1.	DNS 名稱與 IP 地址的限制
    •	在 PrivateLink 或 Gateway Endpoint 出現之前，S3 沒有為 VPC 提供內部的私有 DNS 名稱或私有 IP 地址。
    •	即使你在 VPC 中運行 EC2，當它嘗試解析 [s3.amazonaws.com](http://s3.amazonaws.com/) 時，仍然只能獲得公網 IP。
    2.	S3 是區域（Region）內的全球服務
    •	S3 的 Public Endpoint 是設計為一種「跨 AWS 區域（Region）共享」的服務。這使得它的 Public Endpoint 必須暴露在公網，以便用戶能夠從任何地方（無論是否在 AWS 上）存取 S3。
    3.	VPC 的網路隔離性
    •	VPC 的設計初衷是提供完全隔離的私有網路環境。VPC 裡的資源與外部服務通信時，必須通過 Internet Gateway 或 NAT Gateway，這是早期 AWS 的基本網路架構設計。
    
    PrivateLink/Gateway Endpoint 解決了什麼問題？
    1.	VPC Gateway Endpoint（S3/DynamoDB 特化解法）
    •	AWS 為像 S3 這樣的服務推出了 Gateway Endpoint，允許在 VPC 路由表中直接添加一條到 S3 的內部路由規則。
    •	特點：
    •	你可以直接透過 Private IP 存取 S3，流量完全在 AWS Backbone 網路內傳輸。
    •	適用於大量資料傳輸（例如 Data Lake）。
    2.	PrivateLink（更廣泛的解法）
    •	PrivateLink 則提供了一種更通用的方式，適用於更多 AWS 服務和自有應用程式。
    •	它通過 Interface Endpoint（一個 Elastic Network Interface，ENI）將服務引入你的 VPC 中，所有流量都留在 AWS Backbone 網路內。
    
    總結：在 PrivateLink 或 Gateway Endpoint 出現前的挑戰
    •	流量走公網的根本原因：S3 的 Public Endpoint 設計和 VPC 的網路隔離性，導致 VPC 內的流量無法直接通過 AWS 內部網路訪問 S3。
    •	AWS SDK 的行為：即使使用 AWS SDK，請求仍然透過公共 DNS，無法繞過 Internet。
    •	解決方案的改變：PrivateLink 和 Gateway Endpoint 的出現，讓 VPC 內的資源可以使用 Private IP 安全、高效地訪問 S3 和其他 AWS 服務。
    
- 你確定 AWS SDK 默認是使用 Public DNS 在 internet 存取 S3? 我不太相信耶，如果照你這樣說，那我的 S3 豈不是都要公開？請你上網查證，確保資訊正確
    
    您提出了一个非常重要的问题。在 AWS 中，Amazon S3 的默认端点是公开的，但这并不意味着您的 S3 存储桶或对象默认是公开可访问的。默认情况下，所有 S3 存储桶和对象都是私有的，只有经过授权的用户才能访问。即使通过公共 DNS 名称访问 S3，未经适当的权限配置，外部用户也无法访问您的数据。
    
    在默认情况下，AWS SDK（如 Boto3、AWS CLI 等）使用公共 DNS 名称（如 [s3.amazonaws.com](http://s3.amazonaws.com/)）来访问 S3 服务。这些公共 DNS 名称解析为 Amazon S3 的公共 IP 地址。然而，当您在 VPC 中运行应用程序时，AWS 会自动优化流量路径，使得这些请求通过 AWS 的内部网络进行传输，而不需要经过互联网。这意味着，即使使用公共 DNS 名称，您的数据传输仍然在 AWS 的私有网络中进行，确保了数据的安全性和低延迟。
    
    此外，AWS 提供了 VPC 端点（如 Gateway Endpoint 和 Interface Endpoint），允许您在 VPC 内部通过私有 IP 地址直接访问 S3，而无需通过互联网。这进一步增强了安全性和性能。使用 VPC 端点时，您可以配置私有 DNS，以便在 VPC 内解析 S3 的公共 DNS 名称时，返回 VPC 端点的私有 IP 地址。这使得您的应用程序无需修改代码即可通过私有网络访问 S3。
    
    因此，虽然 AWS SDK 默认使用公共 DNS 名称访问 S3，但在 VPC 环境中，AWS 会自动优化流量路径，确保数据传输在内部网络中进行。同时，使用 VPC 端点和私有 DNS，您可以进一步确保数据传输的私密性和安全性。
    
    有关如何配置 VPC 端点和私有 DNS 的详细信息，您可以参考 AWS 官方文档：
    •	AWS PrivateLink：表示 Amazon S3
    
- 那請問，我在 Lambda 透過 SDK 存取 S3，默認是走 AWS Backbone 網路嗎？不會走到 internet 來存取 S3? 請上網查證，並附上參考資料來源
    
    您提出了一个关键问题，涉及在 AWS Lambda 中通过 AWS SDK 访问 Amazon S3 时，流量路径的具体情况。
    
    默认情况下：
    •	Lambda 函数未连接到 VPC：当您的 Lambda 函数未配置在 VPC 内运行时，它会使用 AWS 的托管网络环境。在这种情况下，通过 AWS SDK 访问 S3 的请求会通过 AWS 的内部网络（AWS Backbone）进行传输，而不会经过公共互联网。这确保了数据传输的安全性和低延迟。
    •	Lambda 函数连接到 VPC：如果您将 Lambda 函数配置为在 VPC 内运行，默认情况下，该函数只能访问 VPC 内的资源。为了使其能够访问公共 AWS 服务（如 S3），您需要配置适当的网络设置。这通常涉及以下两种方法：
    1.	NAT 网关或 NAT 实例：配置一个 NAT 网关或 NAT 实例，允许您的 Lambda 函数通过它们访问互联网，从而访问 S3 的公共端点。然而，这可能会引入额外的延迟和成本。
    2.	VPC 端点（Gateway Endpoint）：为 S3 配置一个 VPC 端点，使您的 Lambda 函数能够通过私有 IP 地址直接访问 S3，而无需经过互联网。这不仅提高了安全性，还减少了延迟和成本。根据 AWS 文档，您可以在 VPC 中创建一个 Gateway Endpoint，以便在不使用 NAT 网关的情况下访问 S3。
    
    参考资料：
    •	在 Lambda 函数中配置 VPC 访问详细说明了如何将 Lambda 函数连接到 VPC，以及如何配置网络以访问所需的 AWS 服务。
    •	为 Amazon S3 设置 VPC 端点提供了有关如何为 S3 配置 VPC Gateway Endpoint 的指导，以实现私有网络访问。
    
    总而言之，默认情况下，AWS Lambda 函数通过 AWS 的内部网络访问 S3，而不会经过公共互联网。然而，当您将 Lambda 函数连接到 VPC 时，需要适当配置网络设置（如 VPC 端点或 NAT 网关），以确保对 S3 的访问仍然保持在 AWS 的内部网络中。
    

延續前一篇 [如何將現有 NLB IPv4-only 架構升級為 Dual-stack](https://www.notion.so/NLB-IPv4-only-Dual-stack-174ea7e0d9d080e5b0e2cbe31f721c6a?pvs=21) 

我們已經有一個 Dual-stack 的 NLB 了。

假如今天有一個需求是「客戶會需要調用我們 API，而且也是用 AWS」，有沒有什麼辦法是不用走 Internet？是直接走 AWS 的骨幹網路 (Backbone) 直接打 API 到我們這裡的 NLB? 而且不需要 VPC Peering 或是 Transit Gateway，避免網路整個打通，東西被看光光，引來不必要風險

答案就是 PrivateLink

![Private Link Architecture Diagram: [https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-privatelink.html](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-privatelink.html)](image.png)

Private Link Architecture Diagram: [https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-privatelink.html](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/aws-privatelink.html)

## Terminology

- **Service Consumer**:
使用服務的一方。Consumer 會在自己的 VPC 中建立 VPC Interface Endpoint 來連接到 Provider 的 Endpoint service。Consumer 必須等待 Provider 接受連接請求後，才能開始使用服務。透過這種方式，**Consumer 可以安全地存取 Provider 的服務，而無需經過公有網路。**
    - **Endpoint**:
    VPC Interface Endpoint (VPCE) 是一個彈性網路介面，具有私有 IP 位址。它作為進入 AWS 服務的進入點，讓 VPC 中的資源可以私密地存取這些服務。在 PrivateLink 架構中，**這是 Consumer 端建立的元件。**
- **Service Provider**:
提供服務的一方。Provider 需要建立 Endpoint service 並將其與 NLB 關聯，然後可以選擇性地允許哪些 AWS 帳戶可以連接到此服務。Provider 負責接受或拒絕來自 Consumer 的連接請求。
    - **Endpoint service**:
    Endpoint service 是由 Service Provider 建立的服務，可以與 Network Load Balancer (NLB) 或是 Gateway Load Balancer (GWLB) 關聯。這個服務允許其他 AWS 帳戶 (Consumer) 通過 VPC Interface Endpoint 連接到 Provider 的服務。

## Section 1: 配置 PrivateLink

### Step 1: [Provider] 創建 Endpoint service

在這個步驟，我們將扮演 Service Provider，我們是提供 Service 的人

**進入 VPC 頁面 > 左側欄 Endpoint services:**

1. 點擊 **Create endpoint service**

![[Provider] Create endpoint service](image%201.png)

[Provider] Create endpoint service

進入 Create endpoint service 頁面後，請依照以下內容進行配置：

1. **Load balancer type:** Network
2. **Available load balancers:** 選擇你的 NLB

其他部分可以保持預設

![[Provider] Create Endpoint service configuration](715d637f-6b78-49cf-8ef2-b48e1cb53492.png)

[Provider] Create Endpoint service configuration

創建好後，複製 **Service name**

![[Provider] Copy service name](image%202.png)

[Provider] Copy service name

### Step 2: [Consumer] 創建 VPC Interface Endpoint (VPCE)

現在我們的身份切換至 **Consumer**

身為 Consumer，我們需要建立一個 **VPC Interface Endpoint (VPCE)**，透過 VPCE 存取到 Service Provider 的服務。

進入 VPC 頁面 > 左側欄 Endpoints:

1. 點擊 **Create endpoint**

![[Consumer] Create endpoint](image%203.png)

[Consumer] Create endpoint

創建 VPC Interface Endpoint，請依照以下內容進行配置：

- **Type:** Endpoint services that use NLBs and GWLBs
- **Service name:** 貼上剛才在 Service Provider 那邊所複製的 Service name

![[Consumer] Create VPC Interface Endpoint configuration](image%204.png)

[Consumer] Create VPC Interface Endpoint configuration

創建好 VPC Interface Endpoint 之後，身為 Consumer，**其實還不能馬上存取 Service，必須等待 Provider 接受你的連線請求才可以使用。**

### Step 3: [Provider] 接受 Acceptance

身份切回 **Service Provider**

回到 VPC 頁面 > 左側欄 Endpoint services:

1. 勾選 **Endpoint service**
2. 進入 **Endpoint connections** 頁籤
3. 勾選 **Endpoint connection** 
4. 展開 **Actions menu** 
5. 點擊 **Accept endpoint connection request**

![[Provider] Accept endpoint connection request](image%205.png)

[Provider] Accept endpoint connection request

### [Consumer] 測試是否可以透過 VPC Interface Endpoint 調用到 NLB

身份切回 **Consumer**

剛才 **Provider** 已經接受連線了，回到 VPC > 左側欄 Endpoint 頁面，檢查一下目前 Endpoint 的 Status：

- 他會停在 **Pending** 狀態一下下，大概 1~2 分鐘，等到變成 **Available** 就 OK 囉！
    
    ![[Consumer] Waiting for endpoint status to be available](image%206.png)
    
    [Consumer] Waiting for endpoint status to be available
    

現在 Endpoint Status 已經是 **Available** 狀態，要測試是否能透過 VPC Interface Endpoint 調用到 Service Provider 的 NLB，首先要先複製 **VPC Interface Endpoint 的 DNS name**

![[Consumer] Copy VPC Interface Endpoint DNS name](image%207.png)

[Consumer] Copy VPC Interface Endpoint DNS name

> 你會注意到 DNS names 提供了兩個 Domain name，下面沒有被紅框框起來的 **Zonal DNS name**，可以保證你的流量一定打到你要的 AZ

關於 AZ 這方面有一個注意事項，可以參考我的筆記: [https://shiun.notion.site/AZ-AZ-95d15fb85b1a4eb2a3cc76e5b3e7bee1?pvs=4](https://www.notion.so/AZ-AZ-95d15fb85b1a4eb2a3cc76e5b3e7bee1?pvs=21)
> 

我現在在 **Consumer** 帳號，透過一台跳板機 SSH 連線到 Private subnet 裡面的一台 Instance

成功連線後，輸入以下指令來存取 VPCE，需要注意， **`curl` 後面實際值換成你剛才複製的 VPC Interface Endpoint DNS name：**

```bash
$ curl vpce-0d3d9cfb73e4becfd-iiv45fgg.vpce-svc-0ceb593b3ba202634.us-west-2.vpce.amazonaws.com

<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
```

透過上面的輸出，這樣就達成「**完全不透過 Internet 就打流量到 NLB**」的架構囉！流量都是在 AWS Backbone 網路中內傳輸。

> **注意：**VPC Interface Endpoint 和 我連線進去的 EC2 Instance 都放在 Private subnet，**我也沒配置 NAT 或是 EIGW**，所以他們是不可能有辦法連到網路的
> 

以下簡單證明沒有連網能力，我先 `ping google.com` 再 `curl` VPC Interface Endpoint

![[Consumer] This consumer instance doesn’t have internet connectivity](image%208.png)

[Consumer] This consumer instance doesn’t have internet connectivity

恭喜！我們已經成功配置好 PrivateLink 囉！

## Section 2: Dual-stack

在 Section 2，將帶大家實現 Dual-stack PrivateLink，讓 Consumer 以及 Provider，同時都可以支持 IPv4 和 IPv6

### [Provider] 為 Endpoint Service 開啟 IPv6 Support

目前身份是 **Service Provider** 

因為當初創建 Endpoint Service 時，沒勾選 IPv6 support，可以觀察下圖的 **Supported IP address type 中僅顯示 IPv4** ，所以用 **IPv6 流量打過去會無法解析**：

![[Provider] This Endpoint service does not support IPv6 address type](image%209.png)

[Provider] This Endpoint service does not support IPv6 address type

以下範例是以 **Consumer** 身份連線進去 Private Instance，利用 `curl -6` 調用 VPC Interface Endpoint 所得到的輸出：

```bash
$curl -6  vpce-0d3d9cfb73e4becfd-iiv45fgg.vpce-svc-0ceb593b3ba202634.us-west-2.vpce.amazonaws.com

curl: (6) Could not resolve host: vpce-0d3d9cfb73e4becfd-iiv45fgg.vpce-svc-0ceb593b3ba202634.us-west-2.vpce.amazonaws.com
```

進入 VPC > 左側欄 Endpoint services:

1. 選取 **Endpoint service** 
2. 展開 **Actions menu**
3. 點擊 **Modify supported IP address types**

![[Provider] Modify supported IP address types](image%2010.png)

[Provider] Modify supported IP address types

在 Supported IP address types 下方選項把 IPv6 也勾選起來：

![[Provider] Enable IPv6 address type](image%2011.png)

[Provider] Enable IPv6 address type

### [Consumer] 使 VPC Interface Endpoint 支持 Dual-stack

目前身份是 **Consumer**

接下來我們也要確保 Consumer 這裡的 VPC Interface Endpoint 也支持 Dual-stack，因此要記住！並不是 Provider 將 Supported IP address types 下的 IPv4, IPv6 都啟用，Consumer 這裡就可以什麼事都不用做就可以用 IPv6 Client 去調用喔，畢竟在 Consumer 這裡，面對的是自己的 “VPC Interface Endpoint”

進入 VPC > 左側欄 Endpoints 

1. 選取 **Endpoint**
2. 展開 **Actions menu**
3. 點擊 **Modify endpoint settings**

![[Consumer] Modify endpoint settings](image%2012.png)

[Consumer] Modify endpoint settings

進入 Modify endpoint settings 頁面：

- **IP address type:** Dualstack
- **DNS options:** Dualstack

![[Consumer] Enable dual-stack support](image%2013.png)

[Consumer] Enable dual-stack support

Save 之後 Endpoint 會進入 **Pending** 狀態，要等 1~2 分鐘直到它變成 **Available** 狀態

當 Endpoint 狀態變成 **Available** 後，SSH 連線 EC2 Instance 再來測試一次吧：

```bash
$ curl -6  vpce-0d3d9cfb73e4becfd-iiv45fgg.vpce-svc-0ceb593b3ba202634.us-west-2.vpce.amazonaws.com

<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
```

> 補充資訊：我這邊剛好沒 Clear CLI，所以可以看到設定成 Dualstack 的前後變化，一開始還無法解析成功，經過上面的重新設定，就成功使用 IPv6 Client 去打流量到 VPC Interface Endpoint 碰到 NLB 唷
> 
> 
> ![image.png](image%2014.png)
> 

恭喜，我們這樣就完成 Dual-stack PrivateLink!

---

## Section 3:  Private DNS name

[Manage DNS names for VPC endpoint services - Amazon Virtual Private Cloud](https://docs.aws.amazon.com/vpc/latest/privatelink/manage-dns-names.html)

Private DNS name 是一個很方便的功能，如果你是 Service Provider，想要配置 Private DNS Name 你必須擁有**域名**，例如: Private DNS name 設定為 [example.com](http://example.com/) 的話，經過 DNS 驗證之後，Consumer 端之後就可以直接用 [example.com](http://example.com/) 來存取這個 Endpoint Service

因為我自己有域名，這裡我就用我自己的域名(shiun.me)來示範

### [Provider] Endpoint Service 設定 Private DNS name

![[Provider] The endpoint service does not support Private DNS name](image%2015.png)

[Provider] The endpoint service does not support Private DNS name

進入 VPC > 左側欄 Endpoint services:

1. 選取 **Endpoint service**
2. 展開 **Actions menu**
3. 點擊 **Modify private DNS name**

![[Provider] Modify private DNS name](image%2016.png)

[Provider] Modify private DNS name

配置 Private DNS name:

1. 勾選 **Associate a private DNS name with the service**
2. 輸入 **Private DNS name:** 我輸入 `demoprivatelink.shiun.me` (請依照你自己擁有的域名自由修改這邊的值)

![[Provider] Set up Private DNS name](image%2017.png)

[Provider] Set up Private DNS name

### [Provider] 域名驗證

創建好之後，**Domain verification status** 會顯示 Pending verification， 這時候我就要根據指示創建一個 TXT Record 證明這個 [shiun.me](http://shiun.me) 域名歸你所管，我們可以從 **Endpoint service detail** 找到 創建 **Record 時**所需要設定的值，如下圖紅框所示

![[Provider] DNS verification details](image%2018.png)

[Provider] DNS verification details

所以我打開我的 DNS 服務配置 TXT Record 

> 注意：下面截圖的畫面，你不見得會跟我一樣
也許你是用 Route53 也許你是用其他域名註冊商本身自己提供的 DNS 服務
> 

![[Provider] Create a TXT Record for DNS verification](image%2019.png)

[Provider] Create a TXT Record for DNS verification

創建完 TXT Record 之後，可能要稍待幾分鐘 (例如 5 mins)，等待 DNS Propagation。

我等了大概 5 mins，回到 VPC 頁面 > 左側欄 Endpoint services:

1. 選取 **Endpoint Service** 
2. 展開 **Actions menu**
3. 點擊 **Verify domain ownership for private DNS name**

![[Provider] Verify domain ownership for private DNS name](image%2020.png)

[Provider] Verify domain ownership for private DNS name

這時候再等一下下，約一分鐘，重新整理一下頁面，就會看到 **Domain verification status 變成 Verified，如果發現還沒 Verified的話，請你：**

1. 再次檢查 **DNS Record 是否有配置錯誤** (Type, value… 等)
2. 再次稍等一下下，可以用 `dig TXT <domain>` 來檢查看看 Answer 是否配置正確
3. 重新 **Verify domain ownership for private DNS name**

![[Provider] Checking the Domain Verification status shows it is verified](image%2021.png)

[Provider] Checking the Domain Verification status shows it is verified

### [Consumer] VPC Interface Endpoint 啟用 Private DNS name

目前身份是 **Consumer**

同理，身為 Consumer 這方，VPC Interface Endpoint 這裡也需要**啟用 Private DNS name 的支持**，可以看到下圖紅框處，**Private DNS name enabled** 顯示 **No，**表示目前還尚未啟用此功能

![[Consumer] The VPC Interface endpoint does not support Private DNS name](image%2022.png)

[Consumer] The VPC Interface endpoint does not support Private DNS name

進入 VPC > 左側欄 Endpoints:

1. 選取 **Endpoint**
2. 展開 **Actions menu**
3. 點擊 **Modify private DNS name**

![[Consumer] Modify private DNS name](image%2023.png)

[Consumer] Modify private DNS name

在 **Modify private DNS name settings** section 中，勾選 **Enable for this endpoint**

![[Consumer] Enable private DNS name for this endpoint](image%2024.png)

[Consumer] Enable private DNS name for this endpoint

Save 之後，會變成 **Pending** 狀態

直到它變成 **Available** 後，我們再次 SSH 連線到 Private Subnet 的 EC2 Instance

`curl -4` 範例輸出：

```bash
$ curl -4 demoprivatelink.shiun.me

<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
```

`curl -6` 範例輸出：

```bash
$ curl -6 demoprivatelink.shiun.me

<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
```

截圖 Private Instance 使用 `curl` 指令分別用 IPv4 和 IPv6 Client 調用 Private DNS name 的執行結果：

![[Consumer] The responses from curl -4 and curl -6 were observed in the consumer's private instance](4e1afe75-3cef-46b2-a2b8-7b6152f88d6e.png)

[Consumer] The responses from curl -4 and curl -6 were observed in the consumer's private instance

從上面的結果可以看到，我們成功使用 Private DNS name 來存取 Service Provider 的 NLB 囉

這裡還可以做一個小實驗，我用自己的筆電打開 Terminal (不是 AWS 上 Consumer VPC 裡面的 EC2 instance 唷)，調用 `curl` 看看:

```bash
$ curl -4 demoprivatelink.shiun.me

curl: (6) Could not resolve host: demoprivatelink.shiun.me
```

```bash
$ curl -6 demoprivatelink.shiun.me

curl: (6) Could not resolve host: demoprivatelink.shiun.me
```

![My Macbook curl -4 and curl -6 responses](image%2025.png)

My Macbook curl -4 and curl -6 responses

我用自己筆電執行的結果是想告訴讀者，PrivateLink 很安全又好用！我們在 AWS 環境外面是真的沒辦法調用到 Endpoint service 唷

---

## 補充 - 透過 PrivateLink 訪問 NLB，那 Service Provider 看得到 Client 原始 IP 地址嗎？

首先，**身為 Consumer** 是透過 VPC Interface Endpoint 去訪問到 NLB 的，我已經找到 VPC Interface Endpoint 的 IP 地址如下:

- **IPv4:** `10.0.133.54`
- **IPv6:** `2600:1f14:3231:3002:d93:b5b3:d78c:2e0a`

![[Consumer] VPC Interface Endpoint IPv4 and IPv6 addresses](image%2026.png)

[Consumer] VPC Interface Endpoint IPv4 and IPv6 addresses

現在我們就來打流量看看吧

**在 Consumer 這方**，我總共輸入了以下指令，也得到相應的 Response:

```bash
[ec2-user@ip-10-0-137-213 ~]$ curl -4  vpce-0d3d9cfb73e4becfd-iiv45fgg.vpce-svc-0ceb593b3ba202634.us-west-2.vpce.amazonaws.com
<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
[ec2-user@ip-10-0-137-213 ~]$ curl -6  vpce-0d3d9cfb73e4becfd-iiv45fgg.vpce-svc-0ceb593b3ba202634.us-west-2.vpce.amazonaws.com
<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
[ec2-user@ip-10-0-137-213 ~]$ curl -4 demoprivatelink.shiun.me
<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
[ec2-user@ip-10-0-137-213 ~]$ curl -6 demoprivatelink.shiun.me
<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
```

**回到 Service Provider 那邊**，我已經有去找到 NLB ENI，其 IPv6 地址為: `2600:1f14:2549:300:17c1:6c5d:ba29:ce1`

我在 Provider 這邊，透過跳板機 SSH 連線到 NLB 背後的 Target Instance 來觀察 Httpd access logs:

```bash
2600:1f14:2549:300:17c1:6c5d:ba29:ce1 - - [08/Jan/2025:04:31:12 +0000] "GET / HTTP/1.1" 200 69 "-" "curl/8.5.0"
2600:1f14:2549:300:17c1:6c5d:ba29:ce1 - - [08/Jan/2025:04:31:12 +0000] "GET / HTTP/1.0" 200 69 "-" "curl/7.88.1"
2600:1f14:2549:300:17c1:6c5d:ba29:ce1 - - [08/Jan/2025:04:31:13 +0000] "GET / HTTP/1.1" 200 69 "-" "curl/8.5.0"
2600:1f14:2549:300:7935:fbac:a7b4:84d5 - - [08/Jan/2025:04:31:16 +0000] "GET / HTTP/1.1" 200 69 "-" "ELB-HealthChecker/2.0"
2600:1f14:2549:300:17c1:6c5d:ba29:ce1 - - [08/Jan/2025:04:31:18 +0000] "GET / HTTP/1.1" 200 69 "-" "curl/8.5.0"
2600:1f14:2549:300:17c1:6c5d:ba29:ce1 - - [08/Jan/2025:04:31:24 +0000] "GET / HTTP/1.1" 200 69 "-" "curl/8.5.0"
```

**可以注意到來源地址都是 NLB 的 IPv6 address！**

**也就是說，Provider 看不到真實來源 IP 地址！**

那怎麼辦？我就是有一些需求需要看到真實來源 IP 地址… 那請繼續往下看，我們會需要 Proxy Protocol 的幫助

### 那如果我現在配置了 Proxy protocol 呢？

關於配置 Httpd Proxy Protocol 支持，這裡先省略，不是本文重點，總之我已經開啟 Proxy Protocol  支持了

於是我再次**回到 Consumer** 的 EC2 Instance 對 VPC Interface Endpoint 發出請求

```bash
[ec2-user@ip-10-0-137-213 ~]$ curl -4  vpce-0d3d9cfb73e4becfd-iiv45fgg.vpce-svc-0ceb593b3ba202634.us-west-2.vpce.amazonaws.com
<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
[ec2-user@ip-10-0-137-213 ~]$ curl -6  vpce-0d3d9cfb73e4becfd-iiv45fgg.vpce-svc-0ceb593b3ba202634.us-west-2.vpce.amazonaws.com
<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
[ec2-user@ip-10-0-137-213 ~]$ curl -4 demoprivatelink.shiun.me
<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
[ec2-user@ip-10-0-137-213 ~]$ curl -6 demoprivatelink.shiun.me
<h1>Hello World from ip-10-0-138-176.us-west-2.compute.internal</h1>
```

**回到 Service Provider 那邊，**觀察 NLB Target instance 中的 Httpd access logs:

```bash
10.0.137.213 - - [08/Jan/2025:04:37:43 +0000] "GET / HTTP/1.1" 200 69 "-" "curl/8.5.0"
2600:1f14:3231:3002:5b88:e457:66c1:43ad - - [08/Jan/2025:04:37:46 +0000] "GET / HTTP/1.1" 200 69 "-" "curl/8.5.0"
10.0.137.213 - - [08/Jan/2025:04:37:50 +0000] "GET / HTTP/1.1" 200 69 "-" "curl/8.5.0"
2600:1f14:3231:3002:5b88:e457:66c1:43ad - - [08/Jan/2025:04:37:52 +0000] "GET / HTTP/1.1" 200 69 "-" "curl/8.5.0"
```

下圖為 Provider 那邊的 NLB Target instance 截圖畫面：

![[Provider] The Httpd access log from the Target instance](image%2027.png)

[Provider] The Httpd access log from the Target instance

根據上述觀察

- `10.0.137.213` 確實是 Consumer Private Instance 的 Private IPv4 address
- `2600:1f14:3231:3002:5b88:e457:66c1:43ad` 也確實是 Consumer Private Instance 的 IPv6 address
- 請見下圖，這是 Consumer 那邊的 Private instance，紅框處就是該 Instance 的 IP 地址
    
    ![[Consumer] Here are the IPv4 and IPv6 addresses for the consumer’s private instance.](image%2028.png)
    
    [Consumer] Here are the IPv4 and IPv6 addresses for the consumer’s private instance.
    

**所以如果有啟用 Proxy Protocol 的話，是可以看到原始來源 IP 的喔！！**

---

## Troubleshooting - [Consumer] 配置 VPC Interface Endpoint 的 SG

如果你沒收到回應，你可以檢查一下 VPC Interface Endpoint 的 SG 是否配置正確，我只有配置一個 Inbound rule 允許 EC2 SG 的流量打進來，至於 Outbound rule 就不需要配置，因為 SG 是 Stateful

![[Consumer] VPC Interface Endpoint Security Group inbound rules](image%2029.png)

[Consumer] VPC Interface Endpoint Security Group inbound rules

## Resources

[AWS PrivateLink Pricing – AWS](https://aws.amazon.com/privatelink/pricing/)

[Manage DNS names for VPC endpoint services - Amazon Virtual Private Cloud](https://docs.aws.amazon.com/vpc/latest/privatelink/manage-dns-names.html)

[AWS PrivateLink 現在支援跨區域連線功能 - AWS](https://aws.amazon.com/tw/about-aws/whats-new/2024/11/aws-privatelink-across-region-connectivity/?utm_source=chatgpt.com)