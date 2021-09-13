---
界: DataBase
門: NoSQL
綱: Redis
tags: ["#Docker","#Note"]
aliases:
  - 
date: 2021-09-07
---

# 基礎概念


## 官方

[Empowering App Development for Developers | Docker](https://www.docker.com)

[Docker Documentation](https://docs.docker.com/)

[GitHub - docker/docker-ce: Docker CE](https://github.com/docker/docker-ce)

---

## DevOps

1.  快速部署上線
2.  打包映象檔案,一鍵測試+發佈
3.  便捷的升級+擴縮容
4.  容器化可以使得開發環境與測試環境高度一致
5.  Docker身為內核級別的虛擬化,可以在一臺物理機上執行多臺容器實例,具備更高效的計算資源利用

---

## Docker 簡介

### 什麼是Docker

通常描述的 `Docker` 是指 `Docker Engine`

Docker Engine 是一個客戶端 - 伺服器的應用程式

Docker是一種輕量級的虛擬化方式

由Docker守護程序 , 一個REST API指定與守護程序互動的介面和一個命令列介面 ( CLI ) 與守護程序通訊(通過封裝REST API)

Docker Engine 從 CLI 中接受 Docker 命令

### Docker 基本組成

-   Docker 主機 [ Host ]
    -   安裝Docker程式的機器
-   Docker 倉庫 [ Registry ]
    -   儲存打包好的軟體映象
    -   倉庫分為公有倉庫和私有倉庫
-   Docker 映象 [ Images ]
    -   軟體打包好后成為映象
-   Docker 容器 [ Container ]
    -   映象啟動后的實例將成為一個容器
    -   容器是獨立執行的一個 或 一組 應用

### Docker 與 操作系統 的區別

傳統的虛擬機器方式提供的是相對封閉的隔離。Docker利用Linux系統上的多種防護技術實現了嚴格的隔離可靠性，並且可以整合眾多安全工具。從 1.3.0版本開始，docker重點改善了容器的安全控制和映象的安全機制， 極大提高了使用docker的安全性。

[區別](https://www.notion.so/57f29bf6b5f541d8b9091f70d95bdd0b)