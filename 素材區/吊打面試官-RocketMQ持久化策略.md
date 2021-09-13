**RocketMQ 採用檔案系統的方式來儲存訊息，訊息的主要儲存檔案包括 CommitLog 檔案、ConsumeQueue 檔案、IndexFile 檔案。**

-   **CommitLog** 是訊息儲存的物理檔案，所有訊息主題的訊息都儲存在 CommitLog 檔案中，每個 Broker 上的 CommitLog 被目前機器上的所有 ConsumeQueue 共享。CommitLog 中的檔案預設大小為 1G，可以動態配置；當一個檔案寫滿以後，會產生一個新的 CommitLog 檔案。所有的 Topic 數據是順序寫入在 CommitLog 檔案中的。
-   **ConsumeQueue** 是訊息消費的邏輯佇列，訊息達到 CommitLog 檔案后將被非同步轉發到訊息消費佇列，供訊息消費者消費，這裡面包含 MessageQueue 在 CommitLog 中的物理位置偏移量 Offset，訊息實體內容的大小和 Message Tag 的 hash 值。每個檔案預設大小約為 600W 個位元組，如果檔案滿了後會也會產生一個新的檔案。
-   **IndexFile** 是訊息索引檔案，Index 索引檔案提供了對 CommitLog 進行數據檢索，提供了一種通過 key 或者時間區間來查詢 CommitLog 中的訊息的方法。在物理儲存中，檔名是以建立的時間戳命名，固定的單個 IndexFile 大小大概為 400M，一個 IndexFile 可以儲存 2000W 個索引。

## **訊息儲存的整體結構**

  

![](https://pic1.zhimg.com/80/v2-f0f5462d41799a90a02d84936b1c8b0c_720w.jpg)

  

RocketMQ 的訊息儲存採用的是混合型的儲存結構，也就是 Broker 單個實例下的所有佇列共用一個日誌數據檔案 CommitLog。這個是和 Kafka 又一個不同之處。為什麼不採用 Kafka 的設計，針對不同的 Partition 儲存一個獨立的物理檔案呢？這是因為在 Kafka 的設計中，一旦 Kafka 中 Topic 的 Partition 數量過多，佇列檔案會過多，那麼會給磁碟的 IO 讀寫造成比較大的壓力，也就造成了效能瓶頸。所以 RocketMQ 進行了優化，訊息主題統一儲存在 CommitLog 中。當然它也有它的優缺點。

-   優點在於：由於訊息主題都是通過 CommitLog 來進行讀寫，ConsumerQueue 中只儲存很少的數據，所以佇列更加輕量化。對於磁碟的訪問是序列化從而避免了磁碟的競爭。
-   缺點在於：訊息寫入磁碟雖然是基於順序寫，但是讀的過程確實是隨機的。讀取一條訊息會先讀取 ConsumeQueue，再讀 CommitLog，會降低訊息讀的效率。

## **訊息發送到訊息接收的整體流程**

1、Producer 將訊息發送到 Broker 后，Broker 會採用同步或者非同步的方式把訊息寫入到 CommitLog。RocketMQ 所有的訊息都會存放在 CommitLog 中，爲了保證訊息儲存不發生混亂，對 CommitLog 寫之前會加鎖，同時也可以使得訊息能夠被順序寫入到 CommitLog，只要訊息被持久化到磁碟檔案 CommitLog，那麼就可以保證 Producer 發送的訊息不會丟失。

  

![](https://pic1.zhimg.com/80/v2-223b8eca06cc5fd9ad28a855d1910b08_720w.jpg)

  

2、CommitLog 持久化后，會把裡面的訊息 Dispatch 到對應的 Consume Queue 上，Consume Queue 相當於 Kafka 中的 Partition，是一個邏輯佇列，儲存了這個 Queue 在 CommitLog 中的起始 Offset，log 大小和 MessageTag 的 hashCode。

  

![](https://pic3.zhimg.com/80/v2-a077178e56a8c2bfb7e7e6f2b14c91a6_720w.jpg)

  

3、當消費者進行訊息消費時，會先讀取 ConsumerQueue，邏輯消費佇列 ConsumeQueue 儲存了指定 Topic 下的佇列訊息在 CommitLog 中的起始物理偏移量 Offset，訊息大小、和訊息 Tag 的 HashCode 值。

  

![](https://pic4.zhimg.com/80/v2-1f4bd9d5f2388c5e779a82bf7d45cd63_720w.jpg)

  

4、直接從 ConsumerQueue 中讀取訊息是沒有數據的，真正的訊息主體在 CommitLog 中，所以還需要從 CommitLog 中讀取訊息。

  

![](https://pic4.zhimg.com/80/v2-e88e30839fd2cb1f28a49af5a0fae053_720w.jpg)

  

**訊息刷盤**  

  

![](https://pic3.zhimg.com/80/v2-4797f54a5c52c71db542d4920ecda7d2_720w.jpg)

  

(1) 同步刷盤：如上圖所示，只有在訊息真正持久化至磁碟后RocketMQ的Broker端才會真正返回給Producer端一個成功的ACK響應。同步刷盤對MQ訊息可靠性來說是一種不錯的保障，但是效能上會有較大影響，一般適用於金融業務應用該模式較多。

(2) 非同步刷盤：能夠充分利用OS的PageCache的優勢，只要訊息寫入PageCache即可將成功的ACK返回給Producer端。訊息刷盤採用後臺非同步執行緒提交的方式進行，降低了讀寫延遲，提高了MQ的效能和吞吐量。