---
title: 'AWS Site-to-Site VPN with Libreswan 建置教學 (Step-by-Step)'
date: 2024-10-22T00:00:14+08:00
draft: false
description: "這篇文章將一步一步教你如何在 AWS 上建立 Site-to-Site VPN"
tags: ["AWS", "Networking", "VPN", "AWS VPC", "Libreswan"]
categories: ["AWS", "Networking"]
keywords:
- AWS
- VPN
- Site-to-Site
- Libreswan
- AWS VPC
- Networking
- AWS VPN
- AWS Site-to-Site VPN
- AWS VPN Connection
- CGW
- VGW
- VPN Server
- VPN Tunnel
- VPN Connection
---
最近正在[準備 AWS ANS 證照](https://shiun.notion.site/AWS-ANS-AWS-Site-to-Site-VPN-126ea7e0d9d0807aad53c41ca301137c)剛好學到 AWS Site-to-Site VPN，也因此實作練習對我來說很重要所以打算寫這篇文章，一方面記錄實作過程一方面留著可以作為教學文章供他人參考

這篇文章將教學如何在 AWS 上建立 Site-to-Site VPN，我們會使用 Libreswan 來配置 VPN Server，這樣就可以在 AWS VPC 和 Data Center (_本文示範會在 N. Virginia Region 中建立 VPC 來模擬 Data Center)_ 之間建立一個安全的 VPN Tunnel，讓兩者可以互相通信

![01-AWS Site-to-SIte VPN](https://github.com/user-attachments/assets/de2d5d9d-97bc-419c-9b5f-5430547d9f3c)

## 一、創建 VPC (Oregon Region)

首先我們先創建架構圖右邊 AWS 的 VPC，因此我切換到 Oregon Region，並且依照下圖配置 VPC 以及 Subnet

![1-1 CreateVPC](https://github.com/user-attachments/assets/7c07b7b3-b95e-4bcb-b404-48cadafc32c5)

![1-2 Create VPC](https://github.com/user-attachments/assets/eda0b350-04a2-478e-9bfb-c5487c104ebe)

## 二、配置 Route Table (Oregon Region)

然後創建 Route Table，並和上一步創建的 Subnet 建立 Association

![2-1 CreatePrivateSubnetRT](https://github.com/user-attachments/assets/1dff8965-86cd-4654-af32-e141e174fd30)

我們要修改 Subnet 的 Association，確保該子網與我們剛才創建的路由表關聯在一起

![2-2 UpdateRTAssociation](https://github.com/user-attachments/assets/4a47f3e1-25d6-430b-8f29-4b38ec565c39)

如下圖配置 Route Table association

![2-3 UpdateRTAssociation](https://github.com/user-attachments/assets/9a5bb41b-384e-4d23-8c07-89f58e105021)

## 三、創建 EC2 Instance (Oregon Region)

接著進入 EC2 Console 頁面，現在要在剛才的 Subnet 啟動一台 EC2 Instance，若沒提到的地方可以保持預設

- 名稱: `EC2-at-AWS`
- AMI 選擇 `Amazon Linux 2023 AMI`
- 然後在 Network settings 處依照下圖配置
  ![3-1 CreateEC2](https://github.com/user-attachments/assets/4f29eca1-5b70-4f9d-b907-698e2e274d9f)

## 四、創建 Data Center ( 這裡以 N. Virginia 的 VPC 來模擬 Data Center )

因為我們沒有實際的資料中心，所以我們就以 N. Virginia Region 中的 VPC 來模擬我們的 Data Center

這裡需要特別注意的是我們要創建 Public Subnet，因為後續在這裡的 EC2 Instance 需要一個 Public IPv4 地址作為我們 Customer Gateway (CGW) 使用

![4-1 CreateVPC](https://github.com/user-attachments/assets/85c7033e-21c9-44ad-a127-4b8c28f619aa)

![4-2 CreateVPC](https://github.com/user-attachments/assets/9b37c1ac-1e4a-4b9f-b7d0-8afad24cf6a2)

## 五、創建 EC2 Instance (N. Virginia Region)

現在需要創建一個 EC2 Instance 作為 VPN Server，這台 Instance 會安裝 VPN 軟體 (Libreswan) 作為 CGW 然後與 AWS VGW 對接形成一個 VPN Tunnel

進入 EC2 Console 頁面，依照下方配置，其他可以保持預設

- 名稱: `VPN-EC2-at-DC`
- AMI 選擇 `Amazon Linux 2023 AMI`
- 創建一個 Key pair 並保存好私鑰，一會我們需要連線至 Instance
- 然後在 Network settings 處依照下圖配置，**一定要記得 Assign Public IP**
  ![5-1 CreateEC2](https://github.com/user-attachments/assets/f952ea9f-a716-4b73-a987-8f6dbc6cbfd8)

## 六、創建 VGW (Oregon Region)

現在回到 VPC Console (Oregon Region) 我們要來配置 VGW

![6-1 CreateVGW](https://github.com/user-attachments/assets/97a5d61d-1d7f-49e6-9749-a6d3c6c0d77c)

創建這個很簡單，如下圖:

![6-2 CreateVGW](https://github.com/user-attachments/assets/b0e66fc3-e93d-42c6-b54a-049325431986)

創建好之後要將其 Attach 到 VPC

![6-3 AttachVGWToVPC](https://github.com/user-attachments/assets/b4e9c7e0-ee99-4f79-bfd7-5ef89b342d3e)

![6-4 AttachVGWToVPC](https://github.com/user-attachments/assets/cffd311d-9c56-4cb0-8df1-4e5afc6a38d8)

![6-5 AttachVGWToVPCCompleted](https://github.com/user-attachments/assets/97fd0aca-9e3d-4d78-b431-6b77a27075af)

## 七、創建 CGW (Oregon Region)

現在我們要創建 CGW，一樣是在 Oregon Region，創建 CGW 最重要的就是 Public IP! 這裡我們是使用別的 Region 的 VPC 來模擬了 Data Center，所以我們要去 N. Virginia Region 複製 EC2 Instance 的 Public IPv4 位址

![7-1 CreateCGW](https://github.com/user-attachments/assets/1c3a6703-097a-4b5e-b6c4-0a56f7587755)

創建好 CGW 如下圖:

![7-2 CreateCGWCompleted](https://github.com/user-attachments/assets/af4b83c6-c772-4c60-9685-d3cd439970f4)

## 八、創建 Site-to-Site VPN connection (Oregon Region)

現在已經創建好 VGW 和 CGW，我們就要來把這兩個 Gateway 連接起來，因此就需要創建 Site-to-Site VPN connection

![8-1 CreateS2SVPNConnection](https://github.com/user-attachments/assets/df02ec7a-cdba-4ccc-adbc-c79aa0894b34)

如下圖配置

![8-2 CreateS2SVPNConnection](https://github.com/user-attachments/assets/d292fb30-6500-4313-9f71-4a0c1d523e27)

創建後需要等待約 3~5 分鐘

## 九、修改 Route Table (Oregon Region)

為了確保 AWS VPC 中的流量知道什麼狀況下要將流量引導至 VGW，所以必須修改 Route Table，如下圖:

![9-1 UpdateRT](https://github.com/user-attachments/assets/0e71d828-be02-423d-aadb-47504ca756f6)

## 十、下載配置檔案

我們準備要連線至 EC2 Instance (N. Virginia) 配置 VPN 軟體了，在這之前，需要先去 Site-to-Site VPN Connection 頁面中 (Oregon) 下載 Configuration file

![10-1 DownloadConfigFile](https://github.com/user-attachments/assets/3aa990e3-6d27-45ca-bed4-b1e92aff49a6)

![10-2 DownloadConfigFile](https://github.com/user-attachments/assets/9f0b79fd-8a23-417f-9780-a1491f912019)

## 十一、配置 VPN Server

準備好你的 .pem 檔案，連線至 EC2

```bash
$ chmod 400 /path/to/your-key.pem
$ ssh -i /path/to/your-key.pem ec2-user@54.32.10.1

   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
```

連線進去後，要來安裝 [Libreswan](https://libreswan.org/)，首先要修改軟體源配置文件，確保我們下載得到 Librwswan

```bash
sudo vi /etc/yum.repos.d/fedora.repo
```

- 指令解釋:
  - 這條指令的功能是用來編輯 Linux 系統中的 **`/etc/yum.repos.d/fedora.repo`** 文件。這是一個典型的場景，適合於 Fedora 或使用 `yum` 軟體包管理器的其他 Linux 發行版。

關於 vi 的操作方式如下，後面不會再重複說明此操作方式:

1. 按下 `i` 進入 INSERT 模式
2. 貼上下方內容
3. 按下 `Esc`
4. 輸入 `:wq`

```bash
[fedora]
name=Fedora 36 - $basearch
#baseurl=http://download.example/pub/fedora/linux/releases/36/Everything/$basearch/os/
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-36&arch=$basearch
enabled=0
countme=1
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=https://getfedora.org/static/fedora.gpg
skip_if_unavailable=False
```

執行以下指令來安裝 Libreswan:

```bash
sudo dnf --enablerepo=fedora install libreswan -y
```

- 指令解釋:
  - 這條指令 **`sudo dnf --enablerepo=fedora install libreswan -y`** 用於在基於 Fedora 的 Linux 發行版上安裝 **Libreswan** 軟體包。

打開 `/etc/sysctl.conf` 然後加上以下內容:

```bash
sudo vi /etc/sysctl.conf
```

然後執行指令 `sysctl -p` ，如下:

```bash
$ sudo sysctl -p

net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.default.accept_source_route = 0
```

- 指令解釋:
  - 指令 **`sysctl -p`** 用於加載並應用系統內核參數的配置文件。修改了 `/etc/sysctl.conf` 後，運行 `sysctl -p` 可以立即應用這些內核參數，無需重啟系統。

檢查 `/etc/ipsec.conf` 檔案最後面的 `include /etc/ipsec.d/*.conf` 前面的 `#` 已被移除，若沒有移除，請使用 vi 開啟該檔案後將 `#` 移除，你可以輸入以下指令來查看 `/etc/ipsec.conf` :

```bash
sudo cat /etc/ipsec.conf
```

創建一個新檔案 `/etc/ipsec.d/aws.conf` :

```bash
sudo vi /etc/ipsec.d/aws.conf
```

接著你務必要參照你在前面下載的 Configuration File 去配置 `/etc/ipsec.d/aws.conf` ，除此之外由於 Libreswan 有一些配置不支援，因此還有一些內容需要小修改，包括:

1. 移除 `auth=esp`
2. `phase2alg=aes128-sha1;modp1024` 修改為 `phase2alg=aes_gcm`
3. `ike=aes128-sha1;modp1024` 修改為 `ike=aes256-sha1`
4. `leftsubnet=<LOCAL NETWORK>` 的 `<LOCAL NETWORK>` 修改為 Data Center 的 CIDR (我的例子是 N. Virginia Region 上的 VPC 來模擬，也就是 `192.168.0.0/16` )
5. `rightsubnet=<REMOTE NETWORK>` 的 `<REMOTE NETWORK>` 修改為 AWS VPC 的 CIDR (我的例子是 Oregon Region 上的 VPC，也就是 `10.0.0.0/16` )

綜合以上，我會在 `/etc/ipsec.d/aws.conf` 中加入以下內容:

```bash
conn Tunnel1
    authby=secret
    auto=start
    left=%defaultroute
    leftid=54.227.52.173
    right=35.81.211.175
    type=tunnel
    ikelifetime=8h
    keylife=1h
    phase2alg=aes_gcm
    ike=aes256-sha1
    keyingtries=%forever
    keyexchange=ike
    leftsubnet=192.168.0.0/16
    rightsubnet=10.0.0.0/16
    dpddelay=10
    dpdtimeout=30
    dpdaction=restart_by_peer
```

創建一個新檔案 `/etc/ipsec.d/aws.secrets` :

```bash
sudo vi /etc/ipsec.d/aws.secrets
```

務必要參照你在前面下載的 Configuration File 去配置 `/etc/ipsec.d/aws.secrets`

以我的為例，我會在 `/etc/ipsec.d/aws.secrets` 加入以下內容:

```bash
54.227.52.173 35.81.211.175: PSK "TFIcmolpZYe4_y0zhjJiSaetWD8F4VZr"
```

終於我們配置好了，可以啟動 Service 囉:

```bash
sudo systemctl start ipsec.service
```

啟動 IPSec Service 後，輸入以下指令檢查狀態:

```bash
$ sudo systemctl status ipsec.service

● ipsec.service - Internet Key Exchange (IKE) Protocol Daemon for IPsec
     Loaded: loaded (/usr/lib/systemd/system/ipsec.service; disabled; preset: disabled)
     Active: active (running) since Mon 2024-10-21 14:15:03 UTC; 27s ago
       Docs: man:ipsec(8)
             man:pluto(8)
             man:ipsec.conf(5)
    Process: 28237 ExecStartPre=/usr/libexec/ipsec/addconn --config /etc/ipsec.conf --checkconfig (code=exited, status=0/SUCCESS)
    Process: 28239 ExecStartPre=/usr/libexec/ipsec/_stackmanager start (code=exited, status=0/SUCCESS)
    Process: 28677 ExecStartPre=/usr/sbin/ipsec --checknss (code=exited, status=0/SUCCESS)
    Process: 28682 ExecStartPre=/usr/sbin/ipsec --checknflog (code=exited, status=0/SUCCESS)
   Main PID: 28693 (pluto)
     Status: "Startup completed."
      Tasks: 2 (limit: 1112)
     Memory: 8.7M
        CPU: 523ms
     CGroup: /system.slice/ipsec.service
             └─28693 /usr/libexec/ipsec/pluto --leak-detective --config /etc/ipsec.conf --nofork

Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: adding UDP interface lo 127.0.0.1:4500
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: adding UDP interface lo [::1]:500
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: adding UDP interface lo [::1]:4500
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: loading secrets from "/etc/ipsec.secrets"
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: loading secrets from "/etc/ipsec.d/aws.secrets"
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: "Tunnel1" #1: initiating IKEv2 connection
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: "Tunnel1" #1: sent IKE_SA_INIT request to 35.81.211.175:500
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: "Tunnel1" #1: sent IKE_AUTH request {cipher=AES_CBC_256 integ=HMAC_SHA1_96 prf=HMAC_SHA1 group=MODP2048}
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: "Tunnel1" #1: initiator established IKE SA; authenticated peer using authby=secret and ID_IPV4_ADDR '35.81.211.175'
Oct 21 14:15:03 ip-192-168-2-57.ec2.internal pluto[28693]: "Tunnel1" #2: initiator established Child SA using #1; IPsec tunnel [192.168.0.0-192.168.255.255:0-65535 0] -> [10.0.0.0-10.0.255.255:0-65535 0] {ESPinUDP=>0xca93ac2d <0x2f788c>
lines 1-28/28 (END)
```

按下 `q` 即可跳出

## 十二、測試是否可以 Ping 到 Oregon Region 上的 Private EC2 Instance

我們已經建立好 Site-to-Site VPN，若配置正常，Data Center 和 AWS 是可以彼此互相通信的

![12-1 CheckTunnel](https://github.com/user-attachments/assets/f0059180-ddef-4194-b665-50f367a1ba6f)

在這裡我們會使用 N. Virginia Region 中的這台 EC2 去 Ping 看看 Oregon Region 上的 Private EC2 Instance，所以請去 Oregon Region 複製其 Private IPv4 address:

```bash
ping 10.0.0.131
```

![12-2 PingPrivateEC2](https://github.com/user-attachments/assets/3ff555cc-43dd-4874-b3e2-17cad1319f51)

可以看到我們成功 Ping 到 Oregon 上的 EC2 Instance 了！

## 十三、Pricing

透過這篇文章，我們學會如何在 AWS 建立 Site-to-Site VPN，最後練習完也別忘記要清理資源避免衍生出不必要的花費。而最後這裡我想稍微介紹一下 Site-to-Site VPN 收費，但最新資訊仍以官方為主 ([**連結**](https://aws.amazon.com/vpn/pricing/))

AWS Site-to-Site VPN 的收費方式如下：

1. **連線費用**：當你建立一個 Site-to-Site VPN 連接時，AWS 會按 **每小時**計費，只要連線處於啟用狀態，無論有沒有數據傳輸都會計算費用。例如，在 Ohio Region，這個連線的費用是 **每小時 $0.05**，一個月持續啟用下來，大約是 **$36** 美元​​。
2. **數據傳輸費用**：數據傳輸會有額外的費用。前 **100 GB 的傳輸是免費**的，但超過 100 GB 後，傳輸費用是 **每 GB $0.09**。例如，如果你一個月傳輸了 1,000 GB，則你需要支付超出免費流量的 **400 GB 傳輸費用（$36**）​。
3. **加速 VPN（選擇性）**：如果你啟用了 **加速 Site-to-Site VPN**（利用 AWS Global Accelerator），則會有額外的費用。除了 VPN 連線費用，還包括每個 Global Accelerator 的費用（每小時 $0.025），以及 Data Transfer Premium（依照地區與傳輸方向計算）​​。

綜合來看，一個基本的 AWS Site-to-Site VPN 連線每月費用可能為 $72，而加速 VPN 的費用更高，可能達到 $123 一個月，視數據量和是否啟用加速功能而定​。

## 相關連結

- [AWS VPN Pricing](https://aws.amazon.com/vpn/pricing/)
- [libreswan](https://libreswan.org/)
