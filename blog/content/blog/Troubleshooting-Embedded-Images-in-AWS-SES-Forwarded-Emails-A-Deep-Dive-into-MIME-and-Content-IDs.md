---
title: '從 AWS SES 轉寄信件無法正確顯示內嵌圖片談起：MIME 內嵌圖片原理與實踐'
date: 2024-08-06T02:25:14+08:00
draft: false
description: "最近用 AWS SES, S3, Lambda 開發了轉寄信件的功能，但是轉寄出去的信件卻無法顯示內嵌圖片，但是我不想把圖片上傳到公開的 S3 Bucket 並依賴 URL 顯示出圖片，因此我仔細研究了 MIME ，透過這篇文章講解電子郵件內嵌圖片的原理 (不依賴外部 URL)。"
tags: ["AWS SES", "AWS Lambda", "AWS S3", "MIME", "Email forwarding", "Base64"]
categories: ["AWS", "Email", "MIME", "Network"]
keywords:
- AWS SES
- Lambda
- S3
- MIME
- Multipurpose Internet Mail Extensions
- email forwarding
- embedded images
- image display issues
- base64 encoding
- email format
- troubleshooting
- inline images
- multipart/related
- 電子郵件轉寄
- 內嵌圖片
---
## 我遇到了什麼問題

最近我用 SES, S3, Lambda 開發了轉寄信件的功能，簡單介紹一下這功能：
假設我就職於公司的開發部門，我們對外有一個信箱是 dev@example.com
當客戶遇到軟體開發上的問題時寄信到 dev@example.com，開發部門的所有同仁就會在自己的信箱收到此客戶寄來的電子郵件，重點是開發部門的同仁們都可以用自己的帳號回覆該客戶

而開發好這功能後，我遇到一個問題 — **轉寄出去的信件沒辦法正確顯示內嵌圖片**

> 關於轉寄信件的程式碼我放在 GitHub 上囉 ([連結](https://github.com/aws-educate-tw/serverless-email-forwarder))，之後有時間會寫一篇 Forward Email 的完整教學，或是也可以關注我的 [Notion](https://shiun.notion.site/Shiun-s-Learning-Journal-ded4cd4a3efd47b2805f2f99dee8f9a1)，我每天都會寫筆記記錄自己的學習過程


### 讓我們直接看例子

首先，我登入私人帳號模擬我是客戶，寄信給 test@aws-educate.tw，下圖是當時寄出去的內容:
![Original-email](https://github.com/user-attachments/assets/1cbf769f-7b8f-46e9-bb8f-240ad7d1fb32)

寄給 [test@aws-educate.tw](mailto:test@aws-educate.tw) 後，SES 會收到信，然後根據我配置的 Receipt rule，原始郵件檔案會被儲存至我指定的 S3 Bucket

延續 [用 SES, S3, Lambda 實現轉寄信件功能](https://www.notion.so/SES-S3-Lambda-18fbed65f6ba41369b89292136e669ec?pvs=21) ，當我嘗試在轉寄出去的信件中內嵌圖片，最後收到轉寄來的信，都會出現下圖的現象：圖片沒辦法內嵌在裡面，倒是跑去附件了

![Forwarded-email](https://github.com/user-attachments/assets/16da38b8-cb36-4d55-b786-6a2f48c8c2ff)

---

## 我起初嘗試的解法

後來下面這篇 Stack Overflow，裡面有人提到 Gmail 還不支援顯示 base64 編碼的圖片 (有待確認，希望有人可以告訴我正確答案)

> As of January 2020, Gmail still does not support base64 encoded images.
>
> Source: [Add embedded image in emails in AWS SES service](https://stackoverflow.com/questions/48587432/add-embedded-image-in-emails-in-aws-ses-service)

因此使用 Data URI: `<img src="data:image/png;base64,iVBORw0KGgoAA" alt="Image">` 嵌入圖片的方式現在可以打消念頭了


接著可能會有人說，那把圖片上傳到 S3，拿到 Object URL，接著在 `<img>` 標籤裡面指定圖片網址不就好，例如: `<img src="https://example.com/assets/image.png" alt="image">`
沒錯，我認為這方法可行，我試過了，但採了無數坑最後失敗告終... 像是把圖片提取出來儲存好，再修改 Message 的內容比我想像中麻煩

沒多久後我想了又想，我只是想要轉寄信件為什麼我要想得那麼複雜，不是應該改個 Message 的 `From`, `To` 和 `Reply-to` 標頭就好了，信件內容就保持原封不動，何必徒增額外功夫先把圖片提取出來，然後想辦法把圖片插入原本的內容中

總而言之，踏過的那些坑，讓我已經對 MIME 研究了一番，所以這篇就來認識一下 MIME 內嵌圖片的原理吧

---

## 認識 MIME 的 boundary

在 [MIME](https://zh.wikipedia.org/zh-tw/%E5%A4%9A%E7%94%A8%E9%80%94%E4%BA%92%E8%81%AF%E7%B6%B2%E9%83%B5%E4%BB%B6%E6%93%B4%E5%B1%95) 訊息中，boundary 是一個用於分隔不同部分的字串。當一封郵件包含多個部分時，例如文字、HTML、圖片或附件，boundary 就用來標記每個部分的開始和結束。

**boundary 的格式：**

boundary 是一個獨特的字串，通常由兩個連字號 (--) 開頭，後面跟著一串隨機字元。例如：

- `-000000000000d37ee1061ed29569`

**如何知道 boundary 的開始和結束：**

1. **Content-Type 標頭：**
    - 在郵件的 `Content-Type` 標頭中，會指定 `multipart` 的類型 (例如 `multipart/mixed`、`multipart/alternative`)，並在 `boundary` 參數中定義 boundary 字串。
    - 例如：`Content-Type: multipart/mixed; boundary="000000000000d37ee1061ed29569"`
2. **分隔符：**
    - 每個部分的開頭都會有一個分隔符，由兩個連字號 (--) 和 boundary 字串組成，後面跟著一個換行符 (\r\n)。
    - 例如：`-000000000000d37ee1061ed29569`
3. **結束標記：**
    - 郵件的最後一個部分結束後，會有一個結束標記，由兩個連字號 (--)、boundary 字串和兩個連字號 (--) 組成，後面跟著一個換行符 (\r\n)。
    - 例如：`-000000000000d37ee1061ed29569--`

**範例：**

```
Content-Type: multipart/mixed; boundary="boundary-string"

--boundary-string
Content-Type: text/plain

This is the email body.

--boundary-string
Content-Type: image/jpeg
Content-Transfer-Encoding: base64

... (base64 encoded image data) ...

--boundary-string--
```

在上面的例子中：

- `boundary-string` 是 boundary 字串
- `-boundary-string` 是每個部分的開頭
- `-boundary-string--` 是郵件的結束標記

---

## 實現內嵌圖片核心原理

### 觀察 Message

> 在 MIME 的上下文中，**message** 是指完整的網際網路消息，在這篇文章的話指的就是一封電子郵件

可以看下方的內容，這是我去 S3 Bucket 把郵件原始檔案 (Message) 下載下來節錄的一個片段，對應了本文開頭圖片的信件內容

```
# 以上略 ....

Content-Type: multipart/related; boundary="000000000000d37ee1061ed2956a"

--000000000000d37ee1061ed2956a
Content-Type: multipart/alternative; boundary="000000000000d37ee1061ed29569"

--000000000000d37ee1061ed29569
Content-Type: text/plain; charset="UTF-8"

test 1047
test 1047test 1047

[image: image.png]

--000000000000d37ee1061ed29569
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: quoted-printable

<div dir=3D"ltr"><div dir=3D"ltr" class=3D"gmail_signature" data-smartmail=
=3D"gmail_signature"><div dir=3D"ltr">test 1047<br></div><div dir=3D"ltr">t=
est 1047test 1047<br></div><div dir=3D"ltr"><br></div><div dir=3D"ltr"><img=
 src=3D"cid:ii_lzeypikh0" alt=3D"image.png" width=3D"176" height=3D"219"><b=
r></div></div></div>

--000000000000d37ee1061ed29569--
--000000000000d37ee1061ed2956a

# 以下略 ...
```

在這裡你要先注意到：我們在外層使用到了 `multipart/related` ，這是 MIME 的一種訊息格式，用於將一個主文件 (main document) 與其相關資源 (related resources) 組合在一起。

以我們現在的情境，使用到 `multipart/related` 就是要拿來內嵌圖片在我們信件內容中

接著下方其實還有一段內容，是我在信件內的想要內嵌圖片:

```
Content-Type: image/png; name="image.png"
Content-Disposition: inline; filename="image.png"
Content-Transfer-Encoding: base64
Content-ID: <ii_lzeypikh0>
X-Attachment-Id: ii_lzeypikh0

iVBORw0KGgoAAAANSUhEUgAAALAAAADbCAIAAABC5oyQAAAQQ0lEQVR4Ae2dz2sbRxvH339hiQkB
45IQ3CSu1IY0IAolhwRXlDqxsB3IQXUP8am02CRg8NU1tLm4PlSJ6KWlF1kWpoUUAkbVLj3pIogR
6CK37sEH51CffElA+KWe8DAzO

# 以下略 ...
```

你可以注意到上方的內容有這段: `Content-ID: <ii_lzeypikh0>` ，這是這張圖片的在這個郵件內的唯一識別符，這個ID可以在 HTML 內容中使用 `cid:` 引用。

因此，實現內嵌圖片的核心除了 `multipart/related` 以外，還有這段 HTML: `<img src="cid:ii_lzeypikh0" alt="image.png" width="176" height="219">`

- 其屬性如下：
  - `src="cid:ii_lzeypikh0"`：表示圖片來源是一個Content-ID，通常在郵件中用來內嵌圖片。
  - `alt="image.png"`：替代文字，在圖片無法顯示時顯示此文字。
  - `width="176"`：圖片的寬度為176像素。
  - `height="219"`：圖片的高度為219像素。

你可能會感到疑惑，通常這裡比較常見就是放圖片 URL，例如 `src="https://my-image.com/image.png"`

然而，這裡的 `src="cid:ii_lzeypikh0"` 是一種特殊的用法，用於電子郵件內嵌圖片。這種用法的 `cid`（Content-ID）表示圖片是電子郵件的一部分，而不是從外部連結加載。這樣的圖片內嵌方法通常用於確保圖片能夠在收件人的郵件客戶端中正確顯示，而不會受到外部圖片加載問題的影響。

---

## 參考資料

- [RFC 2822: Internet Message Format](https://datatracker.ietf.org/doc/html/rfc2822)
- [多用途網際網路郵件擴展 - 維基百科，自由的百科全書](https://zh.wikipedia.org/zh-tw/多用途互聯網郵件擴展)
