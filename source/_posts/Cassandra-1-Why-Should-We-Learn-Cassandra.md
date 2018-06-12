---
title: Cassandra(1)-為什麼我要學Cassandra
tags:
  - cassandra
  - cqrs
  - event sourcing
  - akka
abbrlink: 46d28f28
date: 2018-06-12 23:50:24
---


2, 3年前的某一個專案，我想要使用Akka實現CQRS，查了很久的資料，發現Akka生態系上大家都在推薦使用Cassandra，甚至Akka官方都自己開發一套名為[akka-persistence-cassandra](https://github.com/akka/akka-persistence-cassandra)實現akka-persistence對Cassandra接口。

稍微研究一下，發現Cassandra真的與CQRS天生絕配；CQRS對資料庫主要是用[Event Sourcing(事件源)](https://martinfowler.com/eaaDev/EventSourcing.html)＋[Optimistic lock(樂觀鎖)](https://en.wikipedia.org/wiki/Optimistic_concurrency_control)手法來處理併發問題，

而Cassanra寫的高吐量(除了[Anna](https://databeta.wordpress.com/2018/03/09/anna-kvs/)外，應該還沒有資料庫寫的效能比Cassanra快吧？！)正好可以處理Event Sourcing大部份的操作都在寫資料庫的模式，另一方面，Cassanra又有類似SQL PK(Primary Key)也能輕鬆實現Optimistic Lock；以上兩個特性已經保證了CQRS在Cassanra上可以又快又安全。

另外Cassanrda還有幾個優點:

1. 線性擴展，意即效能可以隨著機器數量增加呈現線性的提昇
1. 地理上的分散，可以設定資料跨資料中心進行複制及備份
1. 無單節點故障，Cassanra是去中心化的集群設計，所以不會Master故障導致整個集群無法使用

> 雖然很多NoSQL都號稱自己可以橫向擴展，但大部份的NoSQL效能表現並不會因為增加了一倍的硬體，而增加一倍的效能； 有些甚至到一定數量後，再怎麼增加硬體也無法提升效能了，如: MongoDB

![Benchmark](https://academy.datastax.com/sites/default/files/benchmark-page-1-load-process.png)


當然Cassandra還是有一些缺點, 但我認為Cassanra最大的問題在於初期的學習曲線太過㞳峭, 在這種情況，很多開發人員就寧可去選擇像MongoDB這種好上手的NoSQL, 畢竟老闆想要的都是越快越好.

會寫這篇系列文的原因, 是之前使用在akka-perisisttence-cassandra時, 很多對Cassandra操作的細節都是框架幫我做掉了, 最近剛好買了一本[Cassandra 技術手冊第二版](http://www.books.com.tw/products/0010760815)(上一版已經是5年前了，好不容易等到新版)，想藉這個機會好好理解Cassandra，也為未來的自己做點筆記.

![Cassandra 技術手冊第二版](https://www.books.com.tw/img/001/076/08/0010760815.jpg)