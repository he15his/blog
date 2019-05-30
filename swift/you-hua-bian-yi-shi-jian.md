\[使用工具定位编译耗时的方法\]\([https://github.com/RobertGummesson/BuildTimeAnalyzer-for-Xcode](https://github.com/RobertGummesson/BuildTimeAnalyzer-for-Xcode)\)

主要包括

1.减少复杂表达式和多元运算，包括??写法

2.多使用 private fileprivate 修饰类和方法，不让编译

到其它无关文件内

3.减少类型判断，定义时直接指定类型

4.少用optional



详见：[https://juejin.im/post/5ad33a086fb9a028cf32ebe1](https://juejin.im/post/5ad33a086fb9a028cf32ebe1)

