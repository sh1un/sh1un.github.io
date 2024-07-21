---
title: 'Dynamic Preload for Performance Optimization'
date: 2024-07-21T23:29:27+08:00
draft: true
description: ""
tags: ["AWS CloudFront", "AWS S3", "Preload", "Frontend Performance Optimization", "Next.js", "React", "Dynamic Loading", "Music Application"]
categories: ["AWS", "CDN", "Frontend Development", "Performance Optimization"]
keywords:
- AWS CloudFront
- AWS S3
- CDN
- Preload
- Dynamic Preload
- Frontend Performance Optimization
- Next.js Performance
- React Preload
- Dynamic Resource Loading
- Music Application Optimization
- Preload in Next.js
- JavaScript Preload
- Web Performance Techniques
- Audio Preloading
- Improve User Experience
- Fast Loading Times
- Frontend Development
---
程式碼在這邊的 `feature/preload` 分支: [https://github.com/sh1un/Nextjs-Musive-app](https://github.com/sh1un/Nextjs-Musive-app/tree/feature/preload)

此專案是一個 Next.js 專案，原作者為 [Ansh Rathod](https://www.linkedin.com/in/ansh-rathod/)，我已經過作者本人授權使用，因此 Fork 過來稍微改了一下。

**我本身不是一名專精於前端技術的開發者**，所以我所修改的 Code 都是由 ChatGPT-4 產生的。我今天這樣做的主要目的是想實驗 preload 是否能做到效能優化，以此來做一個簡單的 PoC。

> 補充一下，這篇文章的內容本來是要放在 「[2024 AWS Educate 陪跑計畫的獎勵課程 - 雲端串流挑戰：復刻 Spotify 的技術旅程](https://www.instagram.com/p/C8UWAZXynuU/?hl=zh-tw&img_index=1)」中教學，此工作坊旨在教學如何利用 CloudFront 優化速度與降低延遲，但礙於當天工作坊的內容太滿已經塞不下了，所以當天工作坊就沒有講到 Preload，如果看到這篇文章，想要學學 CloudFront 歡迎參觀我們當天的教材 ([連結](https://aws-educate-tw.notion.site/20240705-Spotify-6b973b4e8d4d43d5bfb06815732f4e0b))

---

## 什麼是 Preload?

Preload 是一種網頁性能優化技術，讓瀏覽器在頁面加載時預先加載指定的資源，這樣在使用這些資源時可以更快地顯示或播放。這可以減少等待時間，提升用戶體驗。

> 你可以想成：原本你是要“按下去播放按鈕”才會開始下載音樂，現在我們提前幫你下載，你之後“按下去播放按鈕”就會立即播放。

### 舉例

如果你在網頁上有一段影片或音樂，你可以用 preload 告訴瀏覽器提前下載這些文件，這樣當用戶點擊播放按鈕時，影片或音樂就會立即播放，而不是先等待下載。

基本用法：
在 HTML 中，你可以在 `<link>` 標籤中使用 `rel="preload"` 來預加載資源。例如：

```html
<link rel="preload" href="path/to/your/file.mp3" as="audio">
```

這樣，瀏覽器會在頁面加載時預先下載 `file.mp3`，提高用戶播放音樂的速度。

---

## 動態 Preload 的實現思路

每個用戶的歌單都不一樣，所以我們當然不希望把 preload 的 `href` 寫死成一個 URL。

### 解決方案

我們可以通過以下幾個步驟來實現動態的 preload：

1. **API 獲取歌單**：
   伺服器端提供一個 API，根據用戶的請求返回該用戶的歌單。這個 API 返回一個 JSON 結構，其中包含了音樂檔案的 URL 列表。
2. **JavaScript 動態生成 Preload 標籤**：
   使用 JavaScript 在頁面加載後動態地從 API 獲取歌單，並為每個音樂文件創建 preload 標籤，將其添加到頁面的 `<head>` 中。

### 具體實現

> 注意⚠️ 我的寫法並非添加 preload 到頁面的 `<head>`，而是直接用 React 和 JS 本身提供的內建物件來達到同樣效果，詳見本段說明。

因為我們的 Ansh 寫的後端應用有提供這支“取得指定數量隨機音樂 API”，我會以這支 API 來取得隨機歌單：

- `GET /api/songs/random/{songs_count}`
- Response (200)

    ```json
    {
        "success": true,
        "data": [
            {
                "id": 99123,
                "duration": 197.877531,
                "track_name": "Marching Music -The Crusader By John Philip Sousa",
                "src": "https://cdn.pixabay.com/audio/2022/03/21/audio_1a944368a1.mp3",
                "cover_image": {
                    "url": "https://images.unsplash.com/photo-1482954363933-4bed6bbea570?ixid=MnwzODAxMTZ8MHwxfHNlYXJjaHwxMjd8fGx1eHVyeXxlbnwwfHx8fDE2NjgyNjIzOTI&ixlib=rb-4.0.3",
                    "color": "#0c2640"
                },
                "artist_name": "Jane Howe",
                "artist_id": 25232863
            },
            ...
        ]
    }
    ```

先來看一些核心的程式碼，最後會附上完整的 `_app.tsx`。

這段程式碼在 `MyApp` 組件的 `useEffect` 中實現了動態 preload：

```tsx
useEffect(() => {
  // Fetch random songs and preload them
  const fetchAndPreloadSongs = async () => {
    try {
      const { topHits } = await homePageApi.getRandomArtists();
      preloadSongs(topHits);
    } catch (error) {
      console.error("Error fetching random songs:", error);
    }
  };

  const preloadSongs = (songs: Song[]) => {
    songs.forEach((song) => {
      const audio = new Audio();
      audio.src = song.src;
      audio.load();
      audioElements.current.set(song.id, audio);
      console.log(`Preloading song: ${song.track_name}`);
    });
  };

  fetchAndPreloadSongs();
}, []);
```

- **useEffect**
    - 這段程式碼在組件掛載 (mounted) 後執行，確保在頁面加載後立刻執行 preload 邏輯。
    - 空的依賴陣列 `[]` 確保這段程式碼只在組件初始化時執行一次。
- **fetchAndPreloadSongs**
    - 該異步函數使用 `homePageApi.getRandomArtists()` 從 API 獲取隨機的音樂檔案。
    - 獲取音樂後，調用 `preloadSongs` 函數進行 preload。
- **preloadSongs**
    - 該函數接收一個音樂列表，對每個音樂創建一個新的 `Audio` 對象。
    - 設置 `audio.src` 為音樂的 URL，並調用 `audio.load()` 以便瀏覽器開始加載音樂文件。
    - 將創建的 `Audio` 對象存儲在 `audioElements` 這個 `Map` 中，方便以後引用和控制。

`_app.tsx` 完整程式碼：

```tsx
import "../styles/globals.css";
import type { AppProps } from "next/app";
import { Provider } from "react-redux";
import store from "../stores/store";
import NextNProgress from "nextjs-progressbar";
import { useRouter } from "next/router";
import AudioPlayer from "../components/AudioPlayer/AudioPlayer";
import SidebarItem from "../components/sidebarItem";
import Head from "next/head";
import "react-toastify/dist/ReactToastify.css";
import useDetectKeyboardOpen from "use-detect-keyboard-open";
import AddToCollectionModel from "@/components/AddToCollectionModel";
import { ToastContainer } from "react-toastify";
import { useEffect, useRef } from "react";
import homePageApi from "../stores/homePage/homePageApi";
import { Song } from "@/interfaces/Track";

function MyApp({ Component, pageProps }: AppProps) {
  const audioElements = useRef<Map<number, HTMLAudioElement>>(new Map());

  useEffect(() => {
    // Fetch random songs and preload them
    const fetchAndPreloadSongs = async () => {
      try {
        const { topHits } = await homePageApi.getRandomArtists();
        preloadSongs(topHits);
      } catch (error) {
        console.error("Error fetching random songs:", error);
      }
    };

    const preloadSongs = (songs: Song[]) => {
      songs.forEach((song) => {
        const audio = new Audio();
        audio.src = song.src;
        audio.load();
        audioElements.current.set(song.id, audio);
        console.log(`Preloading song: ${song.track_name}`);
      });
    };

    fetchAndPreloadSongs();
  }, []);

  return (
    <Provider store={store}>
      <Head>
        <link
          rel="preload"
          href="/musive-icons.ttf"
          as="font"
          crossOrigin=""
          type="font/ttf"
        />
        <link
          rel="preload"
          href="/ProximaNova/Proxima Nova Reg.otf"
          as="font"
          crossOrigin=""
          type="font/otf"
        />
        <link
          rel="preload"
          href="/ProximaNova/Proxima Nova Bold.otf"
          as="font"
          crossOrigin=""
          type="font/otf"
        />
      </Head>
      <NextNProgress
        color="#2bb540"
        stopDelayMs={10}
        height={3}
        options={{ showSpinner: false }}
      />

      <Component {...pageProps} />
      <AudioPlayerComponent />
    </Provider>
  );
}

function AudioPlayerComponent() {
  const router = useRouter();
  const isKeyboardOpen = useDetectKeyboardOpen();
  return (
    <div>
      <ToastContainer
        position="top-center"
        autoClose={1000}
        hideProgressBar
        newestOnTop={

false}
        closeOnClick
        rtl={false}
        pauseOnFocusLoss
        draggable
        pauseOnHover
        theme="dark"
      />
      <AddToCollectionModel />
      {router.pathname !== "/login" &&
      router.pathname !== "/register" &&
      router.pathname !== "/_error" &&
      router.pathname !== "/" ? (
        <AudioPlayer className={isKeyboardOpen ? "invisible" : "visible"} />
      ) : (
        <div></div>
      )}
      {router.pathname !== "/login" &&
        router.pathname !== "/register" &&
        router.pathname !== "/_error" &&
        router.pathname !== "/playing" &&
        router.pathname !== "/" && (
          <div
            className={`bg-[#121212] hidden mobile:block tablet:block 
      fixed bottom-0 left-0 right-0 w-full pt-2 pb-1 z-20 ${
        isKeyboardOpen ? "invisible" : "visible"
      }`}
          >
            <div className="flex flex-row justify-center ">
              <SidebarItem name="home" label="Home" />
              <SidebarItem name="search" label="Search" />
              <SidebarItem name="library" label="Library" />
            </div>
          </div>
        )}
    </div>
  );
}

export default MyApp;
```

---

## 使用 Preload 前 vs 使用後

### 使用前

請注意，第二首是按下播放按鈕後才開始下載音樂：

![nextjs-no-dynamic-preload](https://github.com/user-attachments/assets/38f7b6d1-a3d3-47a5-9126-f704195bcda7)


### 使用後

![nextjs-use-dynamic-preload](https://github.com/user-attachments/assets/7e219ea3-e105-47d9-b925-af12fff1d55c)

![devTool-screenshot](https://github.com/user-attachments/assets/3ae047b2-751e-4913-be18-5a887fc5da84)

---

## 補充 - 在舊有的機器改成 Preload
這邊內容僅適用於有照著[工作坊內容](https://aws-educate-tw.notion.site/20240705-Spotify-6b973b4e8d4d43d5bfb06815732f4e0b)實作的同學，以下步驟將教學你如何把原有的 Application 套用 Dynamic Preload：
1. 連線上去 EC2
2. `cd Nextjs-Musive-app`
3. `git checkout feature/preload`
4. `TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")`
5. `export MUSIVE_API_URL="http://$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/public-ipv4):4444/api"`
6. `npm install`
7. `npm run build`
8. `pm2 restart nextjs-app`
