---
title: 'Docker 初學常見問題 - CMD vs ENTRYPOINT 兩者的差異與範例'
date: 2024-02-07T20:43:51+08:00
draft: false
description: "很多人在初學 Docker 時，通常都會知道 CMD 和 ENTRYPOINT 基本上可以互換，但又覺得很疑惑既然兩個指令能互換為什麼要提供兩個指令給我們用?
但其實這兩者是有一些差異的，今天這篇文章就是來帶你了解 Dockerfile 中的 CMD 和 ENTRYPOINT"
tags: ["Docker"]
categories: ["Docker"]
keywords:
- Dockerfile
- CMD
- ENTRYPOINT
- docker build
---

## 前言
很多人在初學 Docker 時，通常都會知道 CMD 和 ENTRYPOINT 基本上可以互換，但又覺得很疑惑既然兩個指令能互換為什麼要提供兩個指令給我們用?
但其實這兩者是有一些差異的，今天這篇文章就是來帶你了解 Dockerfile 中的 CMD 和 ENTRYPOINT

## CMD vs ENTRYPOINT

我這邊準備了兩個 Dockerfile，分別使用了 `CMD` 和 `ENTRYPOINT`

```Dockerfile
FROM ubuntu:22.04
CMD [ "echo", "Hello from CMD" ]

```


```Dockerfile
FROM ubuntu:22.04
ENTRYPOINT [ "echo", "Hello from ENTRYPOINT" ]

```

接著 build 了兩個 docker image，分別取名為 `docker-cmd`, `docker-entrypoint`

```shell
$ docker images
REPOSITORY                                   TAG               IMAGE ID       CREATED        SIZE
docker-entrypoint                            latest            fa8ea026cc54   12 days ago    77.9MB
docker-cmd                                   latest            bd73abaca1f4   12 days ago    77.9MB
```


---

### CMD 會被覆寫、ENTRYPOINT 則不會

理應來說我用 `docker-cmd` 這個 image 啟動一個容器，會打印出 Hello from CMD，執行結果確實也是如此:
```shell
$ docker run docker-cmd
Hello from CMD
```

但我現在在 `docker run` 後面加上額外的命令
- 例如我額外加上: echo "I am shiun"
  - 後面額外多加的指令會直接覆蓋掉原本的 `CMD` 指令，`CMD` 中的命令就不會被執行:
```shell
$ docker run docker-cmd echo "I am shiun"
I am shiun
```

同樣在 `docker run` 後面附加額外命令，如果是 `ENTRYPOINT`，則不會被覆寫，但是會傳遞給 `ENTRYPOINT` 作為額外參數
- 例如我額外加上: echo "I am shiun"
  - 這邊額外加上的命令，會被作為參數傳遞給 `ENTRYPOINT` 命令

注意一下！在這邊的上下文，我所指的「參數」是傳達給命令的額外資訊，用於影響該命令的行為。
參數不一定是選項 （像是 `-f` 或 `-p`），它們也可以是其他命令或文本值

```shell
$ docker run docker-entrypoint echo "I am shiun"
Hello from ENTRYPOINT echo I am shiun
```

> 補充說明: 有注意到引號(") 不見了嗎? 引號的目的是確保被包含的字符串作為一個整體被傳遞，而不是被看作多個由空格分隔的獨立參數，就像我們在 Python `print("I am shiun")` ， "I am shiun" 整句被雙引號包裹住，整句會被視為「一個」字串

由上面的執行結果會看到，我們傳入了兩個參數到 ENTRYPOINT
- echo
- I am shiun

因此 `ENTRYPOINT` 最後執行的指令其實是:
```shell
$ echo Hello from ENTRYPOINT echo I am shiun
```

---

### 如果兩個同時使用， CMD 會被作為 ENTRYPOINT 的默認參數

這次我再創建一個新的 Dockerfile，我會在該 Dockerfile 內同時使用 `CMD` 和 `ENTRYPOINT`

```Dockerfile
FROM ubuntu:22.04
CMD [ "echo", "Hello from CMD" ] # CMD 和 ENTRYPOINT 誰前誰後不影響等下的結果
ENTRYPOINT [ "echo", "Hello from ENTRYPOINT" ] # CMD 和 ENTRYPOINT 誰前誰後不影響等下的結果
```

然後我 build 了 Image，取名為 `docker-cmd-entrypoint`

馬上來 run 一個容器看看結果:
```shell
$ docker run docker-cmd-entrypoint
Hello from ENTRYPOINT echo Hello from CMD
```

會發現 `CMD` 原本要執行的指令被當作參數傳遞給 `ENTRYPOINT`

要注意的是！！！ `CMD` 是被當成默認參數傳遞給 `ENTRYPOINT`
注意！！！是**默認**(預設)，也就是如果你沒特別指定，那我就默認使用這個值的概念

讓我們來看一下官方文件的說明: 
> Command line arguments to docker run <image> will be appended after all elements in an exec form ENTRYPOINT, and will override all elements specified using CMD.
>
> 官方文件: https://docs.docker.com/engine/reference/builder/#entrypoint


因此 `ENTRYPOINT` 那邊最後實際執行的指令是:
```shell
$ echo Hello from ENTRYPOINT echo Hello from CMD
```

---

### 同時使用 CMD 和 ENTRYPOINT，在 docker run 指令又加上額外的命令
這邊的情境一樣使用到 `docker-cmd-entrypoint` 這個 Image
但我這次執行 `docker run` 會在後方附上額外的命令

完整的指令會長這樣: `docker run docker-cmd-entrypoint echo "I am shiun"`

這邊可以先停下來思考一下，回顧前面，**特別是官方說明的部分**，猜猜看上面的指令執行結果會是如何
> Command line arguments to docker run <image> will be appended after all elements in an exec form ENTRYPOINT, and will override all elements specified using CMD.
>
> 官方文件: https://docs.docker.com/engine/reference/builder/#entrypoint

執行結果如下: 
```shell
$ docker run docker-cmd-entrypoint echo "I am shiun"
Hello from ENTRYPOINT echo I am shiun
```
由此可知，`docker run` 後面所附加的額外命令，會覆蓋掉 `CMD` 的內容，而且會作為參數傳遞至 `ENTRYPOINT`

---

## 總結
- `ENTRYPOINT`
  - 容器啟動時必須執行的命令
- `CMD`
  - 容器啟動時默認(預設)執行的命令
- `ENTRYPOINT`, `CMD` 同時使用
  - `CMD` 的內容變成 `ENTRYPOINT` **默認**參數
  - `docker run` 後面若有附加命令
    - 會覆蓋掉 `CMD` 的內容，而且會作為參數傳遞至 `ENTRYPOINT`
