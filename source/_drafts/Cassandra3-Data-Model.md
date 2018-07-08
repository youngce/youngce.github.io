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



## 參考資料
* [Apache Cassandra overview](https://blog.dbi-services.com/apache-cassandra-overview/)
* [Understand the Cassandra data model](https://pandaforme.gitbooks.io/introduction-to-cassandra/content/understand_the_cassandra_data_model.html)
* [Difference between partition key, composite key and clustering key in Cassandra?](https://stackoverflow.com/questions/24949676/difference-between-partition-key-composite-key-and-clustering-key-in-cassandra)
