---
title: Cassandra(3)- 資料模型
abbrlink: bd21a762
tags:
 - cassandra
---
還記得在[上一篇文章](90eadf28.html)中，我們嘗試使用`SELECT * FROM users WHERE last_name='yang';`卻得到以下的錯誤訊息
>InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"

雖然CQL與SQL語法非常類似，但Cassandra在資料模型上與傳統關聯式資料庫還是有非常大的不同；Cassandra在設計上開發人員必須先確認查詢條件，再設計資料表結構；相對地，錯誤的資料表結構也將導致無法或很難查詢到想要的資料。

想要做出好的設計，那就必須先了解Cassandra是如何設定它的資料模型。

圖1為Cassandra資料模型的一個範例
![圖1](http://blog.dbi-services.com/wp-insides/uploads/sites/2/2016/10/Cassandra_dataModel.png)

接下來，我們將分三個部份由下而上來說明Cassandra的資料模型
1. Row
1. Table/Column Family(CF)
1. Keyspace

## Row結構

![圖2](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/Screen%20Shot%202016-02-24%20at%2011.46.09.png)
圖2是一筆Cassadnra Row結構，我們可以發現一筆Row可以分成兩個部份:Row Key以及其他的Key-Value欄位

## 參考資料
* [Apache Cassandra overview](https://blog.dbi-services.com/apache-cassandra-overview/)
* [Understand the Cassandra data model](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/understand_the_cassandra_data_model.html)
