---
title: '一個專案需要多個 Dockerfile - 淺談建構上下文 (build context)'
date: 2024-01-20T18:54:37+08:00
draft: false
description: "最近在寫 E2E 測試遇到一個問題，因 E2E 專案中，除了專案本身的 Docker Image 需要 Build 之外，還有多個測試環境的 Image 也要 Build，這造成了我在這個專案上需要創建多個 Dockerfile"
tags: ["Docker", "E2E"]
categories: ["Docker"]
keywords:
- Docker
- E2E
- Blog
- Build
---

最近在寫 E2E 測試遇到一個問題，因 E2E 專案中，除了專案本身的 Docker Image 需要 Build 之外，還有多個測試環境的 Image 也要 Build，這造成了我在這個專案上需要創建多個 Dockerfile

## 發生了什麼問題? 我一開始的錯誤處理方式
菜鳥時期的我，以為 Dockerfile 就是一定得命名為"Dockerfile"，這導致了我沒辦法在專案根目錄下創建三個 Dockerfile，因為會導致命名衝突

那我想出了什麼處理方式？相當簡單，很菜的我，一開始便自然地根據不同環境在專案下創建了不同的目錄，然後在目錄底下存放各自的 Dockerfile
就很類似這種感覺：
```
E2Eproject/
├── testenvironment1/
│   └── Dockerfile
├── testenvironment2/
│   └── Dockerfile
├── requirements.txt
└── Dockerfile
```

接著便接著發生下一個問題 — 錯誤的建構上下文
以其中一個測試環境內的 Dockerfile 為範例，當時我的寫法如下，請特別注意 `COPY ../../ /usr/src/app/` 這行

```Dockerfile
# Use the official Python base image from the DockerHub.
FROM python:3.12

# Set the working directory within the container.
WORKDIR /usr/src/app

# Set the PYTHONPATH environment variable. This is where Python looks for modules.
# It's set to the work directory to allow local modules to be found.
ENV PYTHONPATH /usr/src/app

# Copy the contents of the current host directory into the container's work directory.
COPY ../../ /usr/src/app/

# Install the project dependencies specified in the requirements.txt file.
# The --no-cache-dir option is used to disable the cache and reduce the layer size.
RUN pip install --no-cache-dir -r ../../requirements.txt

# 略 ...

RUN chmod +x /usr/src/app/start-tests.sh

# The command to run the application.
CMD ["sh", "/usr/src/app/start-headless-tests.sh"]

```

會注意到，我使用了 `../../`，會這麼寫是因為專案根目錄(或是說打包所需的檔案)都在此 Dockerfile 所處目錄的上面幾層
接著就是很自然的輸入下方指令，然後就出現 ERROR 了：
```bash
$ cd testenvironment1
$ docker build -t e2e-project-testenv:v1 .
```

```bash
Dockerfile:16
--------------------
  14 |     # Install the project dependencies specified in the requirements.txt file.
  15 |     # The --no-cache-dir option is used to disable the cache and reduce the layer size.
  16 | >>> RUN pip install --no-cache-dir -r ../../requirements.txt
  17 |     
  18 |     # Install additional dependencies for HTML report generation
--------------------
ERROR: failed to solve: process "/bin/sh -c pip install --no-cache-dir -r ../../requirements.txt" did not complete successfully: exit code: 1
Service 'E2E-project' failed to build : Build failed
```

## TL;DR
- Docker 建構上下文就是告訴 Docker 從哪個目錄開始打包檔案，例如：你在執行指令 `docker built .` 這個 "." 就是建構上下文，Docker 會將這個目錄及其子目錄下的所有檔案作為建構上下文打包成 tar 檔案

- Dockerfile 與 build image 的上下文目錄不必強關聯在一起
- 我們在指令中指定一個目錄作為上下文，然後也透過 `-f` 參數指定使用哪個建構檔案，並且名稱可以自己任意命名
    - docker build -t e2e-project-testenv:v1 -f testenvironment1/Dockerfile /myapp


## 先來了解什麼是 Docker 建構上下文


> 推薦文章：[深入理解 docker build 中的建構上下文](https://www.cnblogs.com/FengZeng666/p/16367833.html)



### 什麼是 Docker 建構上下文(build context)
首先，讓我們簡單回顧一下 Docker 的基本概念。Docker 允許您打包應用程式和所需環境到一個稱為 "鏡像"(image) 的容器中。這個鏡像可以在任何安裝了 Docker 的系統上運行。

1. **Docker 建構上下文的概念**：
   - 當您使用 `docker build` 命令建立 Docker 鏡像時，Docker 會將指定路徑下的檔案和目錄打包成一個 tar 檔案。
   - 這個 tar 檔案被稱為 "建構上下文"(build context)。

2. **為什麼需要建構上下文**：
   - Docker 的鏡像是在 Docker 伺服器（通常是遠端伺服器）上建構的。
   - 為了建構鏡像，Docker 伺服器需要訪問到所有必要的檔案，比如源代碼、配置檔案等。
   - 因此，客戶端（您的電腦）會把這些檔案打包成 tar 檔案，然後上傳給伺服器。

3. **Dockerfile 和建構上下文**：
   - Dockerfile 是一個包含了建構鏡像所需步驟的文本檔案。
   - 在 Dockerfile 中，您可以引用建構上下文中的檔案。比如，您可以複製建構上下文中的檔案到鏡像裡，或者執行建構上下文中的腳本。

總結來說，Docker 建構上下文是 Docker 客戶端將建構鏡像所需的檔案打包並傳輸給 Docker 伺服器的過程。這確保了 Docker 伺服器有所有必要的檔案來建構鏡像。

## 為何我一開始的處理方式會出現 ERROR?
關鍵就出在：
- 當我們執行 `docker build -t e2e-project-testenv:v1 .`，Docker 客戶端會先將後面的指定路徑(.) 打包成一個 tar 檔案，傳送給 Docker 伺服器端，接著才會根據 Dockerfile 中所定義的腳本進行構建

什麼意思呢？
- 我當時執行指令 `cd testenvironment1` 接著執行 `docker build -t e2e-project-testenv:v1 .`
- 事實上就是把 testenvironment1 這個目錄以及其子目錄下的所有檔案打包好，傳送至 Docker Daemon
Docker 在 Build 的時候只能取用上下文的檔案，requirements.txt 位於這個目錄的上層(即 E2Eproject 目錄中)，因此它不會被包含在建構上下文中，也就無法在 Dockerfile 中被訪問。


## 更優雅的處理方式
如果建構鏡像時沒有明確指定 Dockerfile，那麼 Docker Client 默認在建構鏡像時指定的上下文路徑下找名字為 Dockerfile 的建構檔案


但事實上，Dockerfile 與 build image 的上下文目錄不必強關聯在一起
我們在指令中指定一個目錄作為上下文
然後也透過 `-f` 參數指定使用哪個建構檔案
並且名稱可以自己任意命名！
並且名稱可以自己任意命名！！
並且名稱可以自己任意命名！！！
這完完全全解惑了我當初菜鳥所以為的「 Dockerfile 就是一定得命名為"Dockerfile"」

例如：
```bash
$ cd E2Eproject
$ docker build -t e2e-project-testenv:v1 -f testenvironment1/Dockerfile .
```

## 參考資料
- [Docker-学习系列25-Dockerfile-中的-COPY-与-ADD-命令.html](https://blog.mafeifan.com/DevOps/Docker/Docker-%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%9725-Dockerfile-%E4%B8%AD%E7%9A%84-COPY-%E4%B8%8E-ADD-%E5%91%BD%E4%BB%A4.html)

- [深入理解 docker build 中的建構上下文](https://www.cnblogs.com/FengZeng666/p/16367833.html)
