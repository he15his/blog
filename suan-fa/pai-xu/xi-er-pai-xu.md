## 希尔排序

# 基本思想

> 　　希尔排序是把记录按下标的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。

简单插入排序很循规蹈矩，不管数组分布是怎么样的，依然一步一步的对元素进行比较，移动，插入，比如\[5,4,3,2,1,0\]这种倒序序列，数组末端的0要回到首位置很是费劲，比较和移动元素均需n-1次。而希尔排序在数组中采用跳跃式分组的策略，通过某个增量将数组元素划分为若干组，然后分组进行插入排序，随后逐步缩小增量，继续按组进行插入排序操作，直至增量为1。希尔排序通过这种策略使得整个数组在初始阶段达到从宏观上看基本有序，小的基本在前，大的基本在后。然后缩小增量，到增量为1时，其实多数情况下只需微调即可，不会涉及过多的数据移动。

　　我们来看下希尔排序的基本步骤，在此我们选择增量gap=length/2，缩小增量继续以gap = gap/2的方式，这种增量选择我们可以用一个序列来表示，{n/2,\(n/2\)/2...1}，称为**增量序列**。希尔排序的增量序列的选择与证明是个数学难题，我们选择的这个增量序列是比较常用的，也是希尔建议的增量，称为希尔增量，但其实这个增量序列不是最优的。此处我们做示例使用希尔增量。

![](https://images2015.cnblogs.com/blog/1024555/201611/1024555-20161128110416068-1421707828.png)

  



#### 代码实现

```
public static void sort(int[] arr) {
    int length = arr.length;
    //区间
    int gap = 1;
    while (gap < length) {
        gap = gap * 3 + 1;
    }
    while (gap > 0) {
        for (int i = gap; i < length; i++) {
            int tmp = arr[i];
            int j = i - gap;
            //跨区间排序
            while (j >= 0 && arr[j] > tmp) {
                arr[j + gap] = arr[j];
                j -= gap;
            }
            arr[j + gap] = tmp;
        }
        gap = gap / 3;
    }
}
```

可能你会问为什么区间要以 gap = gap*3 + 1 去计算，其实最优的区间计算方法是没有答案的，这是一个长期未解决的问题，不过差不多都会取在二分之一到三分之一附近。