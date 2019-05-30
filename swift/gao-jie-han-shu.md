[链接](https://www.infoq.cn/article/swift-brain-gym-high-order-function)

高阶函数特点
* 接受一个或多个函数当作参数
* 把一个函数当作返回值

### map
我们可以用 map 方法来对数组元素进行某种规则的转换，例如：

```Swift
let arr = [1, 2, 4]
// arr = [1, 2, 4]
 
let brr = arr.map {
    "No." + String($0)
}
// brr = ["No.1", "No.2", "No.4"]

```

### flatMap
flatMap较 map 不同点
* 可将高阶数组转化成一阶
* 可过滤掉 optional 数组中的 nil

### filter
我们可以利用 filter 方法，来对数组元素进行某种规则的过滤，例如：

```
let arr = [1, 2, 4]
// arr = [1, 2, 4]
 
let brr = arr.filter {
    $0 % 2 == 0
}
// brr = [2, 4]
```

### reduce
我们可以用 reduce 方法，来对数组元素进行某种规则的求和（不一定是加和）。

```
let arr = [1, 2, 4]
// arr = [1, 2, 4]
 
let brr = arr.reduce(0) {
    (prevSum: Int, element: Int) in
    return prevSum + element
}
// brr = 7
let crr = arr.reduce("") {
    if $0 == "" {
        return String($1)
    } else {
        return $0 + " " + String($1)
    }
}
// crr = "1 2 4"
```

### forEach
即使是以前最简单的遍历，我们也可以用高阶函数的写法，将遍历需要的操作，以函数参数的形式传入 forEach 方法中，例如：
```
let arr = [1, 2, 4]
arr.forEach {
    print($0)
}
```