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

下圖為Cassandra資料模型的一個範例
![https://blog.dbi-services.com/apache-cassandra-overview/](http://blog.dbi-services.com/wp-insides/uploads/sites/2/2016/10/Cassandra_dataModel.png)

接下來，我們將分三個部份由下而上來說明Cassandra的資料模型
1. Row
1. Table/Column Familiy(CF)
1. KeySpace



## Row結構
