---
title: 淺談Akka(1)- Akka與Actor Model
tags:
  - scala
  - akka
  - actor
categories:
  - Akka
abbrlink: ff8df8fd
date: 2018-06-10 09:28:48
---
# 前言
Akka是實現Actor Model的一個框架，同時支援Java及Scala，它天生的高效，容錯，分散等特性，目前Java生態系中也有許多開源專案使用Akka，例如: Apache Spark, Apache Flink等。

其實Actor Model的應用最一開始出現在1987年愛立信公司開發的Erlang中，其中最成功的案例就是愛立信公司使用Actor Model在他們服務中，達到20年可用時間99.9999999% (nine-nines, 九個九)。

究竟Actor Model有哪些特性，讓我們看下去...

# Actor Model
## 優美的高併發問題(Concurrent)解決方案

1. 若你有一個`Counter`物件，主要任務是將請求的值加總起來，
1. `Counter`的狀態(state)儲存在外部資料庫中，可能是 Reids 或SQL
1. 每次狀態(state)更新後都需要寫回資料庫
`Counter`程式碼可能如下，
```scala
class Counter(){
    def getState:Int= //由外部資料庫取得狀態
    def add(i:Int) = {
        val newState=getState+i
        //...
        // newState寫回資料庫
    }
}
```
請問如何保證在多個線程同時操作時，`Counter`的狀態不會出錯？
這是我在面試後端工程師最常問的問題之一.

併發問題基本上有幾個解決方案:
1. 鎖(Lock)
    悲觀鎖或樂觀鎖
1. 佇列(Queue)
    使用佇列保證請求的順序性

而Actor Model中採用佇列的方式來解決併發問題，每個Actor都有自己的Mailbox，
任何人都無法直接取得Actor的狀態，只能透過發送Message的方式請求Actor回應，每個Actor在該消息(Message)未處理完之前是不會處理下一個消息的，那些等待被處理消息就會被暫時存放在Mailbox中。如下圖，
![Actor Model](https://blog.scottlogic.com/rdoyle/assets/ActorModel.png)

Actor Model的設計除了解決併發問題外，因為要取得Actor的狀態唯一的方法就是向該Actor發送Message請求，
所以Actor之間並不彼此依賴，也使得每個Actor之間是鬆耦合，提高架構的擴展及可維護性。


> 注意
> 目前Akka中發送消息機制預設為`最多傳送一次(At most once)`，所以消息可能會丟失, 這個問題我們會在後面討論在什麼情況下會發生，以及該如何解決消息丟失的問題

其實Actor Model設計也反映在現實生活中，例如你向同事(Actor)詢問工作上要用的資料，你可能會選擇口頭詢問或是寄Mail給他(發送Message)，當他有空處理你的消息後，自然就會給你回應了，容忍的時間長短，則取決於事務急迫性了。

## Let's it crash
Actor Model的另一個特點是它的高容錯性，它的概念是程式運行時總會遇到非預期的錯誤，例如: 網路斷線，連不到資料庫等

那我們應該如何處理錯誤，應該取決於業務內容及錯誤的嚴重性，該讓它掛的時候就讓它掛吧(Let's it crash)。

在Actor Model中每個建立的Actor(除了ActorSysmtem之外)都有一個Supervisor，當Actor發生錯誤時，Supervisor會根據你的錯誤策略做出相對應該的處置，

就像是在現實職場上，你若犯了某錯誤，懲處將會由你的主管(Supervisor)決定，

在這裡，我們先暫時將Supervisor提供的4種錯誤處理策略想像成職場的語言, 會比較好理解，如下表

| Akka        | 職場           | 
| ------------- |:-------------:| 
| Resume      | 再給一次機會 | 
| Restart    | 換人做做看      |  
| Stop | 離職| 
| Escalate | 回報上級，請求上級指示| 

# Akka
Actor Model在Scala v2.10前是預設支持, 不需要下載額外的套件，當初[Martin Odersky](https://twitter.com/odersky)只想證明Scala的語法是可以很容易實現Actor Model，但在Scala v2.11後決定將Actor Model移出Scala, 改用Akka同名公司全力發展，並延伸更多的應用。

以目前來說，你幾乎可以只使用Akka完成你所有的後端服務，以下是我幾個常用的Akka框架，也會這個淺談系列中一一介紹。

### [Akka Persistence](https://doc.akka.io/docs/akka/2.5/persistence.html)
Akka本身的Actor模組是不處理資料持久化，所以發展出Akka Persistence專門負責持久化Actor狀態；若你有聽過[CQRS](https://martinfowler.com/bliki/CQRS.html)或[Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)這類的設計架構，或許你可以考慮使用Akka Persistence來實現

### [Akka Http](https://doc.akka.io/docs/akka-http/current/?language=scala)
顧名思義，你可以建立一個純HTTP接口服務，Akka Http底層是以Akka Stream實現的.

### [Akka Stream](https://doc.akka.io/docs/akka/2.5/stream/index.html)
類似RX，將數據以Stream方式處理，整個結構分為Source(來源)，Sink(輸出)，Flow(過程)三個部份，建構出自己的資料處理流。目前也是Akka中最受大家注目的框架之一.

# 總結
程式設計一直想要模仿真實世界的行為，因為真實世界常見的行為可以存在這麼久，主要的原因就是它合理且自然. 

Actor Model是我目前最喜歡的架構，某種程度上它實現在真實世界某些合理自然的溝通模式，希望透過淺談Akka系列可以讓更多人了解Actor Model及Akka，享受不用於傳統CURD的設計架構。

若有錯誤，請不吝指教.

