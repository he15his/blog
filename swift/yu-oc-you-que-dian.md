[地址](http://www.cocoachina.com/ios/20190520/26989.html)

## 1、swift和OC的共同点：

 - OC出现过的绝大多数概念，比如引用计数、ARC（自动引用计数）、属性、协议、接口、初始化、扩展类、命名参数、匿名函数等，在Swift中继续有效（可能最多换个术语）。

 - Swift和Objective-C共用一套运行时环境，Swift的类型可以桥接到Objective-C（下面我简称OC），反之亦然

## 2、swift的优点：

 - swift注重安全，OC注重灵活

 - swift注重面向协议编程、函数式编程、面向对象编程，OC注重面向对象编程

 - swift注重值类型，OC注重指针和引用

 - swift是静态类型语言，OC是动态类型语言

 - swift容易阅读，文件结构和大部分语法简易化，只有.swift文件，结尾不需要分号

 - swift中的可选类型，是用于所有数据类型，而不仅仅局限于类。相比于OC中的nil更加安全和简明

 - swift中的泛型类型更加方便和通用，而非OC中只能为集合类型添加泛型

 - swift中各种方便快捷的高阶函数（函数式编程） (Swift的标准数组支持三个高阶函数：map，filter和reduce,以及map的扩展flatMap)

 - swift新增了两种权限，细化权限。open > public > internal(默认) > fileprivate > private

 - swift中独有的元组类型(tuples)，把多个值组合成复合值。元组内的值可以是任何类型，并不要求是相同类型的。

## 3、swift的不足：

 - 版本不稳定

 - 公司使用比例不高，使用人数比例偏低

 - 有些语法其实可以只规定一种格式，不必这样也行，那样也行。像Go一样禁止一切（Go有点偏激）耍花枪的东西，同一个规范，方便团队合作和阅读他人代码。