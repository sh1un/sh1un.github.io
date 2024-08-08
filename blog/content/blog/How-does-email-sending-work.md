---
title: '以真實世界寄信來比喻，理解電子郵件寄送流程'
date: 2024-08-08T21:27:44+08:00
draft: false
description: "這篇文章透過郵局寄信的過程進行比喻，了解電子郵件的寄送流程"
tags: ["SMTP", "Email", "Network", "DNS"]
categories: ["Network", "DNS"]
keywords:
- Email sending process
- SMTP
- How email works
- Mail Transfer Agent (MTA)
- Email server
- 電子郵件寄送過程
- 電子郵件如何運作
---
因為我最近在開發一些寄信的系統，因此理解電子郵件寄送的流程和原理是很重要的，為了方便理解，所以我會用真實世界中我們去郵局寄信來比喻電子郵件的寄送流程，這麼做是為了幫助理解，所以比喻那部分很難做到 100% 完全和技術細節對齊

## 用真實世界在郵局寄信，理解電子郵件寄送的流程

![Email-sending-process](https://github.com/user-attachments/assets/1629a1db-c2ac-4f83-a0b0-b4cb2fa43e80)
_圖片取自: [維基百科](https://zh.wikipedia.org/wiki/%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6#%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%BD%AF%E4%BB%B6%EF%BC%88%E9%83%A8%E5%88%86%EF%BC%89)_


為了更好地理解電子郵件的寄送過程，我們可以把它比作郵局寄信的過程。假設 Alice 給 Bob 寫了一封感謝信，然後她將信件帶到郵局處理寄信手續。接下來，我們來看看具體過程:

1. **創建信件**
    - **郵局寄信**: Alice 寫好感謝信，裝進信封，寫上寄件人和收件人的地址，貼上郵票。
    - **電子郵件**: 你在郵件客戶端 (例如: Outlook、Gmail) 或透過 API 建立一封郵件，內容包含寄件人 (例如: `alice@example.tw`) 、收件人 (例如: `bob@demodomain.tw`) 、主題、內文及其他可能的附件。

2. **信件寄送**
    - **郵局寄信**: Alice 把信帶到郵局，然後找郵局的櫃台人員。
    - **電子郵件**: 當你點擊「寄送」後，郵件客戶端會將這封郵件交給寄件人的郵件伺服器，稱為 SMTP 伺服器 (Simple Mail Transfer Protocol) 。

3. **SMTP 伺服器處理**
    - **郵局寄信**: 郵局的櫃台人員會檢查地址是否正確，然後進行分類。
    - **電子郵件**: 寄件人的 SMTP 伺服器會驗證寄件人的身份，檢查信件格式是否正確，並根據收件人的域名 (例如: `demodomain.tw`) 查找相應的 MX 記錄 (Mail Exchange) 來確定收件人的郵件伺服器位置。
    > 像如果對方沒有設定 MX Record，寄信過去就會出現以下 ERROR，找不到對應的 MX Record
![MX-Record-Not-found](https://github.com/user-attachments/assets/58ee45f1-57fa-463d-a7b1-24947657a1f6)

4. **DNS 查詢**
    - **郵局寄信**: 郵局櫃台人員會根據地址查找對應郵遞區號。
    - **電子郵件**: 寄件人的 SMTP 伺服器會透過 DNS (Domain Name System) 查詢收件人域名的 MX 記錄，以獲取負責接收郵件的伺服器的 IP 位址。

5. **信件傳輸**
    - **郵局寄信**: 郵差將信件從一個郵局運送到下一個郵局，直到信件到達收件人的郵局。
    - **電子郵件**: 寄件人的 SMTP 伺服器使用 TCP/IP 協定，將信件傳輸到收件人的 SMTP 伺服器。這個過程可能會經過多個中繼伺服器，但最終會到達收件人的 SMTP 伺服器。

6. **接收與存儲**
    - **郵局寄信**: 收件人的郵局收到信件後，會將信件投遞到 Bob 的信箱中。
    - **電子郵件**: 收件人的 SMTP 伺服器收到信件後，會根據收件地址將郵件存儲到相應的收件人郵箱中。這個過程可能涉及過濾垃圾郵件、病毒掃描等安全檢查。

7. **通知收件人**
    - **郵局寄信**: Bob 會定期檢查他的信箱，找到 Alice 的信件。
    - **電子郵件**: 收件人的郵件伺服器將新郵件存儲到收件人的收件箱中後，會通知收件人的郵件客戶端 (如 Outlook、Gmail) 有新的郵件到達。這通常是透過 IMAP (Internet Message Access Protocol) 或 POP3 (Post Office Protocol) 完成的。

8. **收件人查看信件**
    - **郵局寄信**: Bob 拿到信件後，拆開信封閱讀信件內容。
    - **電子郵件**: 收件人的郵件客戶端接收到新郵件通知後，會在用戶界面中顯示這封新郵件。當 `bob@demodomain.tw` 開啟郵件客戶端 (例如: Gmail)時，就能看到你寄送的信件。

通常，第一個電子郵件伺服器並不是電子郵件的最終目的地。當伺服器收到來自用戶端的電子郵件後，會與另一台郵件伺服器進行重複的 SMTP 連線程序。第二台伺服器也會執行相同的程序，直到電子郵件最終到達收件人電子郵件提供者所控制的郵件伺服器上的收件匣。

![image](https://github.com/user-attachments/assets/c9ac3b9b-7409-4fa8-9043-47b62aaf340d)
_圖片取自: [維基百科](https://zh.wikipedia.org/zh-tw/%E9%82%AE%E4%BB%B6%E4%BC%A0%E8%BE%93%E4%BB%A3%E7%90%86)_

可以將此程序比喻成一封信件從寄件人傳送到收件人的過程。郵差無法直接將寄件人的信件交給收件人，信件可能會先到一個中繼所，然後運送到另一個城鎮的郵局，然後再運送到下一個郵局，如此反覆...直到信件到達收件人。同樣地，電子郵件透過 SMTP 從一個伺服器傳送到另一個伺服器，直到它們到達收件人的收件匣。

這個過程看似複雜，但實際上在數秒鐘內就能完成。整個電子郵件系統的設計使得信件能快速、安全地從一個地方傳送到另一個地方。

## 參考資料

- [簡易郵件傳輸通訊協定 SMTP 是什麼？ | Cloudflare](https://www.cloudflare.com/zh-tw/learning/email-security/what-is-smtp/)
- [訊息傳輸代理 - 維基百科，自由的百科全書](https://zh.wikipedia.org/zh-tw/邮件传输代理)
- [電子郵件](https://zh.wikipedia.org/wiki/%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6#%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%BD%AF%E4%BB%B6%EF%BC%88%E9%83%A8%E5%88%86%EF%BC%89)
