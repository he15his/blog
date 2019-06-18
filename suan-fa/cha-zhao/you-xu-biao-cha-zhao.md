有序查找指先对有序的数据进行查找操作



几种查找的时间复杂度都是 O(logn)

## 1.二分查找

每次查找操作选择中间的元素为基准点，与被查找元素比较，然后再选取左或者右边继续进行下次查找

基准点选取为`mid = low + (high - low) / 2`



```c
int BinarySearch(int array[], int n, int value)
{
    int left = 0;
    int right = n - 1;
    
    while (left <= right)  //循环条件，适时而变
    {
        int middle = left + ((right - left) >> 1);  
        if (array[middle] > value)
            right = middle - 1;  //right赋值，适时而变
        else if (array[middle] < value)
            left = middle + 1;
        else
            return middle;
    }
    return -1;
}
```



## 2.插值查找

插值查找主要是基准点的选取为`mid = low + ((key - a[low]) / (a[high] - a[low])) * (high - low)`,减少查找的次数。


对于表长较大，而关键分布又比较均匀的查找表来说，插值查找算法的平均性能比折半查找要好得多

## 3.斐波那契查找

斐波那契数据为：0，1，1，2，3，5，8，13，21，34 这样的数列

选取基准点的位置使用斐波那契数列对应的下标  `mid = lod + F[k-1] - 1`



二分查找是进行加法与除法运算

插值查找是进行四则运算

斐波那契查找是最简单的加减法运算, 在海量数据查找中，这种差别会影响最终查找效率