---
title: Scala中的函數式編程-模式匹配1(Pattern Matching)
tags:
  - scala
abbrlink: 964a19b7
date: 2018-06-11 00:21:46
---
![Pattern Matching](https://s3-eu-west-1.amazonaws.com/www.voxxed/2017/09/PatternMatching.jpg)
Pattern Matching算是Scala中的一大特色, 比起以往使用的if else或是switch-case有著更強大且優美的能力，
以下我將Scala的Pattern Matching使用方法分為4個部份，由簡單到因難依序介紹。
## 傳統方法
以下代碼示範了一個類似if else的例子，我們可以藉這個例子來了解patten matching的結構，
```scala
def toYesOrNo(choice: Int): String = choice match {
    case 1 => "yes"
    case 0 => "no"
    case _ => "error"
  }

// toYesOrNo(1)=>"yes"
// toYesOrNo(0)=>"no"
// toYesOrNo(33)=>"error"
```

 `_`表示任意的情形, 若不想使用`_`也可以寫成以下的形式,
```scala
def toYesOrNo(choice: Int): String = choice match {
    case 1 => "yes"
    case 0 => "no"
    case whaterver => "whaterver "
  }
```

## 類型模式
 
除了判斷數值外，也可以判斷類型，如下，
```scala
def f(x: Any): String = x match {
    case i:Int => "integer: " + i
    case _:Double => "a double"
    case s:String => "I want to say " + s
  }

// f(1) → “integer: 1″Typ
// f(1.0) → “a double”
// f(“hello”) → “I want to say hello”
```
## Functional approach to pattern matching

以下是一個Factorial的傳統遞迴方法
```scala
def fact(n: Int): Int ={
    if (n == 0) 1
    else n * fact(n - 1)
  }
```
改以pattern matching來實現, 又會如何呢??
```scala
def fact(n: Int): Int = n match {
    case 0 => 1
    case n => n * fact(n - 1)
  }
```
我們甚至可以限定n的範圍，如
```scala
		case n if (n>0)=>n * fact(n - 1)
```

## 模式匹配與集合

來試試一個集合加總的遞迴實現, 我們可能會寫出以下的代碼
```scala
def length[A](list : List[A]) : Int = {
    if (list.isEmpty) 0
    else 1 + length(list.tail)
  }
```
看起來沒什麼問題, 但在pattern matching下有更酷的寫法,

```scala
def length[A](list : List[A]) : Int = list match {
    case _ :: tail => 1 + length(tail)
    case Nil => 0
  }
```

 `Nil` 代表集合為空時,

 `_::tail` 應該被理解成, “a list with whatever head followed by a tail.”我的解釋是能將一個list拆分為list.head與list.tail

## 總結
最後，我們用以下的例子總結我們在這篇文章提到的所有技巧
```scala
def parseArgument(arg : String, value: Any) = (arg, value) match {
  case ("-l", lang) => setLanguageTo(lang)
  case ("-o" | "--optim", n : Int) if ((0 < n) && (n <= 5)) => setOptimizationLevelTo(n)
  case ("-o" | "--optim", badLevel) => badOptimizationLevel(badLevel)
  case ("-h" | "--help", null) => displayHelp()
  case bad => badArgument(bad)
}
```
了解Pattern Matching後，在下一篇我們將與```case class```結合，看看能產生什麼樣的火花。
