---
title: Cassandra(2)-Docker上安裝Cassandra
tags:
  - docker
  - cassandra
abbrlink: 90eadf28
date: 2018-06-14 17:36:26
---


## 預先準備環境
使用Docker安裝的好處是目前各大OS都有支援的Docker版本
，另一方面面對不熟悉的新東西，只有些參數設定不小心設錯可能會把自己的電腦弄壞，但使用Docker也是不用擔心這個問題了，要是真的不小心玩壞了，只要重啟另個新的Container就好了，而且啟動一個新的Container也是相當快速的。

請預先安裝好Docker, 安裝方法可以參考官網，無論你的作業環境是Windows(要玩OpenSource還是花錢買台MacBook, 或重灌成Linux系統吧XD), MacOS, 或是Linux都官網都詳䀆說明。


[Docker官方安裝文件](https://docs.docker.com/install/)

## 安裝Cassandra
我自己使用的是Cassandra官方提供的Docker Image

執行以下Script，第一次可能會比較慢，因為Docker為先從Dockerhub下載Image下來。
```bash
docker run --name some-cassandra -d cassandra:3.11.2
```
我啟動了一個名為`some-cassandra`的容器，Image是`cassandra:3.11.2`表示我們要啟動是Cassandra的版本3.11.2也是到今天為止Dockerhub上提供最新版本號了。

你可以使用`docker ps`確認我們的`some-cassandra`是否啟動，以下圖
![some-cassandra的狀態是Up表示正常運行中](https://i.imgur.com/lVkpN3i.png)

## Cassandra的基本操作
### 啟動cqlsh
由於我們剛剛啟動的`some-cassandra`並沒有把Port連接到本機上，所以我們要進入容器中操作Cassandra，並啟動`cqlsh`,請執行以下指令
```
docker exec -it some-cassandra /bin/bash
cqlsh # 在container中執行
```
成功應該會出現如下圖一樣的畫面，表示你成功進入cqlsh了。

![](https://imgur.com/JSuis7U.png)

### cqlsh 的增刪改查(CURD)
在使用cqlsh之前有幾個Cassandra的名詞要稍微介紹一下，下面的表是跟傳統的關聯式資料庫名詞做對應，可以讓大家更好理解，

|Cassandra|關聯式資料庫|
|---|---|
|cql|sql|
|Keyspace|Database|
|Table|Table|

所以接下來的流程我們將會先建立一個名為KeySpace，再建立該KeySpace中的一張名為`users`的Table.

> 你可以在cqlsh中按下TAB鍵，它會自動補齊或提示你可是是想要的保留字； 另外方向鍵的上下可以讓你找回之前執行過的命令。

```sql
CREATE KEYSPACE IF NOT EXISTS my_keyspace 
    WITH replication = {
        'class': 'SimpleStrategy',
        'replication_factor': 1
        };
DESCRIBE my_keyspace
USE my_keyspace ; 
```
這裡有幾點需要注意:

1. cql是不分大小寫的，它會將你的命名(如KeySpace，Table等)全部轉為小寫，所以建議你使用蛇形命名規則(snake_case，即不同的單字間使用`_`隔開)
1. `replication`中的兩個屬性`class`及`replication_factor` 是Cassandra多台集群的設定，目前我們還不需要使用它，所以選用最簡單的方式，但在正式的生產環境就不能這樣設定了
1. `DESCRIBE` 類似SQL中的`SHOW`

接著我們執行以下命令建立一張`users`表, 並寫入一個名為`mark yang`的user

```sql
CREATE TABLE users ( first_name text PRIMARY KEY , last_name text );
INSERT INTO users (first_name , last_name ) VALUES ( 'mark','yang');

```

你可以使用以下語法查看users表中的資料，甚至計算該表中的資料筆數，cql語法跟sql非常的相似

```sql
SELECT * FROM users;
SELECT COUNT(*) FROM users;
SELECT * FROM users WHERE first_name='mark';
```

> 如果你嘗試使用`SELECT * FROM users WHERE last_name='yang';` 你會得到一個錯誤訊息，這個問題與Cassandra的資料結構有關，我們將在下一篇文章中討論。

最後讓我們來看看cql的更新及刪除的語法吧
```sql
-- 更新first_name為mark的資料
UPDATE users SET last_name = 'yang2' WHERE first_name ='mark' ;
-- 刪除first_namt為mark的所有資料
DELETE FROM users WHERE first_name ='mark'
-- DROP及TRUNCATE都是刪除整張資料表的語法
DROP TABLE users ;
TRUNCATE users;
```

我們現在已經把Cassandra的環境搭建起來，也知道一些cqlsh的基本操作，
下一篇我們將一起來看看Cassandra中資料儲存的方式。

