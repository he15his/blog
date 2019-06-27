类簇有特殊内存处理，其它的一般在大括号结束自动加上release,加方法生成的对象会自动加上autorelease,在最近的一个autoreleasepool结束时释放，没有的话就是在runloop结束的一个autoreleasepool

###  {#autoreleasepool}



