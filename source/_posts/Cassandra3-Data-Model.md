---
layout: drafts
title: Cassandra(3)- 資料模型
tags:
  - cassandra
abbrlink: bd21a762
date: 2018-07-20 00:54:37
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
圖2是一筆單純的Cassadnra Row結構，我們可以發現一筆Row可以分成兩個部份:Row Key以及其他的Key-Value欄位。
### Row Key
又稱為主鍵(Primary Key)，通常由一個欄位組成，就像關聯式資料庫一樣，不同Row的主鍵是不能重複的，但是若Insert相同的主鍵，Cassandra並不會出現錯誤訊息，而是直接該主鍵的Row做Update的動作；你可以試著執行以下的CQL，你不會像SQL一樣得到寫入重複主鍵的錯誤訊息。
```sql
-- 建立users表，first_name為主鍵
CREATE TABLE my_keyspace.users (
    first_name text PRIMARY KEY,
    last_name text
);
-- 寫入一筆mark yang
INSERT INTO users (first_name, last_name) VALUES ( 'mark','yang') ;
-- 寫入一筆mark lin，主鍵為mark
INSERT INTO users (first_name, last_name) VALUES ( 'mark','lin') ;
```

若只能使用一個欄位做為主鍵，在真實的開發環境似乎很不實用，所以Cassandra也支持多個欄位的主鍵設定, 下方CQL建立了一個`employees`表，分別有部門(department), 年紀(age), 收入(salary)，姓名(first_name, last_name)，並以department, first_name, age等三個欄位作為主鍵。

```sql
-- 建表
CREATE TABLE employees (
    department text,
    age int,
    salary double,
    first_name text,
    last_name text,
    PRIMARY KEY (department, first_name, age)
);
```

這樣以多個欄位建立主鍵的方式在Cassandra中被稱為組合鍵(Composite Key)或複合鍵(Compound Key)。

一個複合鍵包含了一個分區鍵(Partition Key)以及一組叢集欄位(Cluster Columns): 分區鍵負責決定該筆Row屬於哪一個分區並要儲存在Cassandra集群中的哪一個節點，而叢集欄位則負責分區內的資料排序方式; 這樣設計的好處擁有相同分區鍵的資料會集中在同一個節點，當對叢集欄位做排序或是分組的操作時(你無法對一般欄位做任何的排序，分組以及條件查詢)，也只是在同一個節點運算，並不需要跨節點運算, 所以分區鍵必須要好好設計，避免資料過份集中在某一個節點上。 接著我們來看看以下幾個查詢範例，其中也包含錯誤的查詢語法，
```sql
-- 寫入資料
INSERT INTO employees (department , first_name , age , last_name , salary ) 
        VALUES ( 'RD', 'mark',  30, 'yang', 10000);
INSERT INTO employees (department , first_name , age , last_name , salary ) 
        VALUES ( 'RD', 'jack',  22, 'li',   8000);
INSERT INTO employees (department , first_name , age , last_name , salary ) 
        VALUES ( 'HR', 'tina',  21, 'chang',3000);
INSERT INTO employees (department , first_name , age , last_name , salary ) 
        VALUES ( 'HR', 'kim',   23, 'lin',  5000);
INSERT INTO employees (department , first_name , age , last_name , salary ) 
        VALUES ( 'HR', 'winnie',24, 'li',   1000);

-- 查詢所有結果 OK!!
SELECT * FROM employees ; 
-- 查詢條件: 分區鍵等於RD OK!!
SELECT * FROM employees WHERE department='RD';
-- 查詢條件: 分區鍵大於RD FAIL！！
-- 必須指定分區鍵，Cassandra才知道要去哪個節點找資料
SELECT * FROM employees WHERE department>'RD';
-- 查詢條件: 分區鍵等於RD，叢集欄位first_name等於jack OK！！
-- 可以嘗試改為first_name>'jack'
SELECT * FROM employees WHERE department='RD' AND first_name='jack' ;
-- 查詢條件: 分區鍵等於RD，叢集欄位age大於0 FAIL！！ 
-- 必須照建立叢集欄位的順序下條件查詢
SELECT * FROM employees WHERE department='RD' AND age>0 ;
-- 查詢條件: 分區鍵等於RD，一般欄位salary大於0 FAIL！！ 
-- 不能直接對一般欄位下條件查詢
SELECT * FROM employees WHERE department='RD' AND salary>0 ;
```

那如何辨別欄位是分區鍵，叢集欄位又或是一般欄位呢？？
最簡單方式可以由CQL介面中欄位的顏色辨別，如圖3, 以`employees`表為例，分區鍵為`department`, 叢集欄位為`first_name`及`age`;

![圖3](https://i.imgur.com/kCMUTiE.png)


另外一種方式是一個網頁工具[CQL Data Modeler](http://www.sestevez.com/sestevez/cassandradatamodeler/)輸入你的Table Schema即可得到一個視覺化圖形Row結構，如圖4
![圖4](https://i.imgur.com/mlCVSC1.png)

> 若主鍵只包含單一欄位，該欄位則為分區值且無叢集欄位，而CQL Data Modeler不支援無叢集欄位的Schema

> Cassandra支持多個欄位為分區鍵，可以在建表時使用`PRIMARY KEY ((department, first_name), age)`即表示department, first_name為分區鍵

### 欄位
欄位是Cassandra中最小單位，除了欄位名稱及值外，在定義欄位時還需要記錄值的類型，可以到[Apache Cassandra官方網頁](http://cassandra.apache.org/doc/latest/cql/types.html)或是[DataStax](https://docs.datastax.com/en/dse/6.0/cql/cql/cql_reference/refDataTypes.html)查詢目前支持的資料類型，大部份類型都與關聯式資料庫相似，但在Cassandra又比傳統關聯式資料庫多了一些類型，像是:
* [集合類型](http://cassandra.apache.org/doc/latest/cql/types.html#collections): 可以細分為Map，Set以及List三種集合類型
* [自定義類型](http://cassandra.apache.org/doc/latest/cql/types.html#user-defined-types): 你可以使用CQL語法 `CREATE TYPE your_type_name(...)` 建立自己的資料類型； 由於Cassandra並不支持`JOIN`, 因此你也可以考慮使用自定義類型包裝成一個巢狀的資料結構

但除了這些外，每個欄位還有兩種非常重要的屬性: 時間標記及存活時間(TTL，Time to live)
#### 時間標記
每次寫入資料到Cassandra資料庫時，每個欄位的更新時間都會成為該欄位的時間標記，當同一欄位在更新時間發生衝突，通常會以最新的更新操作為主; 舉例來說，為了系統的高可用性，我們通常會將資料分散在不同的Cassandra節點上，若其中一台節點損壞系統仍可正常運作，但當這台損壞的節點重新啟動，資料必定與其他節點不一致，所以Cassandra內部將會以擁有最新的時間標記的更新操作來維持資料的一致性。
> Cassandra集群中每台節點的機器時間非常重要，若節間點的機器時間不一致，可能會使資料不一致或是資料遺失，建議使用網路校時以及使用UTC時區

我們可以使用內建的`WRITETIME()`函數查看該欄位的時間標記，但有一點要注意，主鍵或集群欄位不能使用 `WRITETIME()`查看時間標記的

```sql
-- OK
SELECT last_name, WRITETIME(last_name)  FROM employees ;
-- Fail
SELECT department, WRITETIME(department)  FROM employees ;
```
#### 存活時間(TTL, Time to live)
Cassandra支持資料的存活時間，也就是說超過了指定時間該資料會自動被刪除，但Cassandra更有彈性的是它可以為每個欄位單獨設定TTL，另外可以使用`TTL()`函數查看剩餘時間, 與`WRITETIME()`函數相同，`TTL()`僅作用於一般欄位，對於主鍵或集群欄位均不能使用；來看看一下的語法，我們試著查看所有last_name欄位的剩餘時間，

```sql
SELECT first_name, last_name, ttl(last_name) FROM employees WHERE department='HR' AND first_name='tina' AND age=21;
```
從結果可以發現`ttl(last_name)`的值均為null, 表示該欄位資料永久保留，接下來加上TTL設定看看

```sql
-- 將tina的last_name改為test ttl, 並只保留30秒
UPDATE employees USING TTL 30 SET last_name = 'test ttl' FROM employees WHERE department='HR' AND first_name='tina' AND age=21;

-- 查詢tina的欄位變化
SELECT first_name, last_name, ttl(last_name) FROM employees WHERE department='HR' AND first_name='tina' AND age=21;
```
當你重複查詢tina的欄位變化時，會發現`ttl(last_name)`不再是null, 而是一個數字並不斷的減少，當減少至null的`last_name`也同時變成了null，表示這次更新的資料保留了30秒就被清除了。

既然`TTL()`函數不能使用在主鍵及集群欄位上，那我們又該如何做Row的TTL呢？比較簡單的方法就是在INSERT時就加入TTL，試著執行以下語法

```sql
-- 寫入資料 jimmy
INSERT INTO employees (department , first_name , age , last_name , salary ) 
        VALUES ( 'RD', 'mark',  30, 'yang', 10000);
-- 查詢jimmy的欄位變化
SELECT first_name, last_name, ttl(last_name) FROM employees WHERE department='HR' AND first_name='jimmy' AND age=35;
```
重複查詢jimmy欄位變化時，除了`ttl(last_name)`一樣的不斷地減少外，當減少至null時，整個Row結果都查詢不到，表示這筆資料已經被清除了.



## 參考資料
* [Apache Cassandra overview](https://blog.dbi-services.com/apache-cassandra-overview/)
* [Understand the Cassandra data model](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/understand_the_cassandra_data_model.html)
* [Difference between partition key, composite key and clustering key in Cassandra?](https://stackoverflow.com/questions/24949676/difference-between-partition-key-composite-key-and-clustering-key-in-cassandra)
