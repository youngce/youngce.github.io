---
title: 淺談Akka(1)- Akka與Actor Model
date: 2018-06-10 09:28:48
tags: [scala,akka,actor]
categories: [Akka]
---
# 前言
Akka是實現Actor Model的一個框架，同時支援Java及Scala，它天生的高效，容錯，分散等特性，目前Java生態系中也有許多開源專案使用Akka，例如: Apache Spark, Apache Flink等。

其實Actor Model的應用最一開始出現在1987年愛立信公司開發的Erlang中，其中最成功的案例就是愛立信公司使用Actor Model在他們服務中，達到20年可用時間99.9999999% (nine-nines, 九個九)。

究竟Actor Model有哪些特性，讓我們看下去...

# Actor Model
## 併發問題(Concurrent)
1. 若你有一個`Counter`物件，主要任務是將請求的值加總起來，
1. `Counter`的狀態(state)儲存在外部資料庫中，可能是 Reids 或SQL
1. 每次狀態(state)更新後都需要寫回資料庫
程式碼如下，
```scala
class Counter(){
    def getState:Int= //由外部資料庫取得當前狀態
    def add(i:Int) = {
        val newState=getState+i
        //...
        // newState寫回資料庫
    }
}
```
請問如何保證在多個用戶同時操作時，`Counter`的狀態不會出錯？

這是我在面試後端工程師最常問的問題之一

![Actor Model](https://blog.scottlogic.com/rdoyle/assets/ActorModel.png)
