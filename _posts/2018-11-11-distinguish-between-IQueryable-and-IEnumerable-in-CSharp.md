---
layout: post
title:  "IQueryable和IEnumerable以及AsEnumerable()和ToList()的区别"
categories: 编程
tags: CSahrp Linq
author: 小飞
excerpt: 在C#中使用 Linq to sql 时，经常会不知道怎么使用 IQueryable和IEnumerable 这两种类型，也常常搞混 AsEnumerable()和ToList() 这两个方法。本文就分析下它们之间的区别是什么，以及分别适用于什么情况。
---

* content
{:toc}

**注意：本文背景为 Linq to sql 。文中`ie`指代`IEnumerable`，`iq`指代`IQueryable`。**

##  IQueryable 和 IEnumerable 的区别
* IQueryable  
**延时**执行；扩展方法接受的是Expression(必须要能转成sql，否则报错)

* IEnumerable  
**延时**执行；扩展方法接受的是Func(C#语法)

## AsEnumerable() 和 ToList() 的区别
* ToList()  
**立即**执行，加载数据到内存中。

* AsEnumerable()  
**延时**执行，真正使用时才加载数据。  

对IQueryable对象使用AsEnumerable()后，仍然是延时执行，不过此时对象本质已经变了。  

前面已经说了`IEnumerable的扩展方法接受的是Func(C#语法)`，当ie对象(iq转变)真正使用时，会有2个步骤：  

1. 它会把**iq对象(转变之前的)**的扩展方法翻译成sql语句，查询出数据加载到内存中，变为ie对象；  
2. 此时再把**ie对象(转变之后的)**的扩展方法，使用C#求解，得到最终结果。 

例如：  

iq对象的Skip、Take方法，会被翻译成sql，在数据库里执行取出最终结果。  

而ie对象的Skip、Take方法，则会**取出全部数据到内存中，在内存中执行Skip、Take**，会耗费大量资源。


## 误区

* 误区1：对 iq对象 和 ie对象 使用foreach时，对于循环的每项都要查询数据库。
> **错误！**  
> foreach针对的是**数据集整体对象**（枚举器？）。当使用foreach时，不管是iq对象还是ie对象，它们都是**查询数据库一次，然后开始循环，直至循环结束**。不过，当后续再次使用iq对象或ie对象的具体数据时，它们仍然会再次查询数据库。


## 结论

假设我们把*最终数据之前的数据*称为**中间数据**，那么：
1. 当**中间数据只是作为条件筛选**，需要的只是层层筛选之后的最终数据时，应该继续使用IQueryable，防止加载不必要的数据到内存中。
2. 当**存在中间数据，且中间数据被重复使用**时，应该使用IQueryable.ToList()立即加载到内存里使用(都被重复使用了，应该叫做最终数据了吧..)；
3. 如果**中间结果无用，且想对IQueryable对象使用Func(C#语法)的扩展方法**，应该使用IQueryable.AsEnumerable()转成IEnumerable对象，进行后续操作。


## 参考

1. [LINQ查询中的IEnumerable\<T\>和IQueryable\<T\>](https://www.cnblogs.com/long-gengyun/p/3929900.html)
2. [LINQ使用细节之.AsEnumerable()和.ToList()的区别](https://www.cnblogs.com/Mainz/archive/2011/04/08/2009485.html)
3. 建议29：区别LINQ查询中的IEnumerabl\<T\>和IQueryable\<T\> - 陆敏技《编写高质量代码改善C#程序的157个建议》
