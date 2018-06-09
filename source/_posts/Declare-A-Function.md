---
title: Scala - 函數宣告
date: 2018-06-09 22:26:57
tags: [scala, fp, function]
---



## 前言
Scala是具備函數式編程(Functional Programming)特性的程式語言，因此如何使用函數是學習Scala必要的過程，在這個章節中我們將會了解
- 如何宣告函數?
- Closures(大陸將Cloures翻譯成閉包)
- 使用val宣告函數

## 如何宣告函數？
```scala
def function_name(p1:T1, p2:T2,...,pN:TN):T={
	//do something...
}
```

其中 `def` 為保留字，用來宣告函數；`p1,..., pN`表示此函數所需要的參數；`T`表示回傳的結果類型，在Scala中是不需要使用`return`這個保留字，因為它會自動判斷最後一行程式碼的回傳的類型, `T`亦可以省略，讓Scala自動判斷類型，如
```scala
def function_name(p1:T1, p2:T2,...,pN:TN)={
	//do something...
}
```
注意！！參數類型是不可省略的！！！
接下來讓我們來看看以下幾個簡單的實例吧！！！
#### Example 1
定義一個平方(square)函數.
**input:**	2
**output:** 4
```scala
def square(x:Int):Int= ???

```
#### Answer 1
```scala
def square(x:Int):Int= x*x
```
#### Example 2
定義一個判斷是否為奇數的函數.
**input:**	2
**output:** false
```scala
def isOdd(x:Int):Boolean= ???
```
#### Answer 2
```scala
def isOdd(x:Int):Boolean= x%2==1
```

**補充:** `%`為取餘數的運算子，如`x%2`表示x除以2的餘數

## Function Closures
closures在計算機中可以理解成變量的有效範圍，如

```scala
def A(a:T)={
	val b=a
	println(b)	
	a
}
println(b)	
```
在函數`A`中我們宣告了變量`b`，`b`的有效範圍只會在函數A中，我們最後一行寫的`println(b)`將會導致程式出錯，對編譯器來說這時候的b是還未被定義的。

在Scala中特別的地方是可以在函數內再另行宣告一個子函數，而子函數也具備closures的性質，
```scala
def func(a:T1)={
	def subFunc(b:T2)=???
  
	//do something
	
}
	
```

`subFunc`的有效範圍僅在`func`中而己，一旦離開func使用subFunc，編譯器將會出現錯誤。
現在讓我們來看看下面幾個例子
#### Example 3
定義一個平方和(sum of squares)函數.
**input:**	2, 3
**output:** 13
```scala
def sumOfSquares(x:Int,y:Int):Int= ???

```
#### Answer 3
```scala
//Answer3.1
def square(x:Int):Int= x*x 
def sumOfSquares(x:Int,y:Int):Int= square(x)+square(y)

//Answer3.2
def sumOfSquares(x:Int,y:Int):Int= {
	def square(x:Int):Int= x*x
	square(x)+square(y)
}

```
Answer3.1與Answer3.2兩個方法均可正確執行，差別在於
Answer3.1在```sumOfSquares```中呼叫**外部**的```square```，其他函數**可以重複使用**square函數；
Answer3.2則是在```sumOfSquares```中**宣告**了```square```，若其他函數需要使用square，則須**重新宣告**

## 使用val宣告函數
在Scala中有另一種使用`val`宣告函數的方式，讓我們直接來看看讓怎麼做
```scala
val func:T1=> TRes = x=>//do something!!
```
其中`=>`念作to，

以上程式碼`val func:T1=> TRes`表示宣告**一個**參數類型為`T1`(只有一個參數！！！)，回傳類型為`TRes`的函數，且此函數名為`func`，
注意！！`T1=> TRes`不可以省略，編譯器是無法判斷函數類型的！！

另外`x=>//do something!!`，`x`是可隨意命名的`=>`後面可以實現此函數功能。
為了讓大家更清楚這種宣告方式，我們將以上的Example1~2改用`val`的方式來實現吧！！

```scala
// Example 1
val square:Int=>Int= x=>x*x
// Example 2
val isOdd:Int=>Boolean = x=> x%2==1
```
看到這裡大家可能會有個疑問-像sumOfSquares這種多參數的函數，應該怎麼使用val宣告呢？？？
基本上Scala對於多參數函數的宣告採用了與`Tuple`相同的宣告方式(Tuple的使用我們將會在集合的章節做詳細的討論)，如下
```scala
val func:(T1,T2)=> TRes = (x1,x2)=>//do something!!
```
特別注意```(T1,T2)```表示為第一個參數(x1)類型為T1, 第二個參數(x2)類型為T2; 超過2個以上的參數則以此類推，如下
```scala
val func:(T1,...,TN)=> TRes = (x1,...,xN)=>//do something!!
```
注意！！Scala中Tuple最大長度為22, 但在函數中參數是否能超過22個我自己沒測試過，也無法保證，但可以確定的是一個函數不應該有這麼多個參數，你應該可以拆成更多個小函數去分擔運算邏輯。

現在讓我們來看看Example 3該如何轉換吧！
```scala
// Example 3
val sumOfSquares:(Int,Int)=> Int= (x,y)=>square(x)+square(y)
```
## 總結
在這個章節我們已經了解可以使用`def`及`val`兩種方法在Scala中宣告函數，
其中最大的不同在於`def`可以省略回傳類型，但`val`用於宣告函數時，則必須指定函數類型，如`Int=>Boolean`。
另外，Scala是可以在函數中再宣告子函數，同樣具備Closure的性質。
在一個章節，我們將會討論遞迴函數，一起來體會Scala中優美的函數式編程。
