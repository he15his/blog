
**open**

open 修饰的 class 在 Module 内部和外部都可以被访问和继承
open 修饰的 func 在 Module 内部和外部都可以被访问和重载（override）

**Public**

public 修饰的 class 在 Module 内部可以访问和继承，在外部只能访问
public 修饰的 func 在 Module 内部可以被访问和重载（override）,在外部只能访问

**Final**

final 修饰的 class 任何地方都不能不能被继承
final 修饰的 func 任何地方都不能被 Override

总结：
现在的访问权限则依次为：open，public，internal，fileprivate，private。

[原链接](https://www.jianshu.com/p/a5c2b91b23e7)

