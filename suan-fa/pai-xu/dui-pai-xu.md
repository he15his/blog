1

```c
//堆排序
void heapAdjust(int *a, int i, int n) {
    //不是叶子结点才进行排序
    if (i <= n/2) {
        int left = i*2+1; //左结点
        int right = i*2+2;//右结点
        int max = i;//临时变量
        
        //找出左右结点最大的值
        if (left < n && a[left] > a[max]) {
            max = left;
        }

        if (right < n && a[right] > a[max]) {
            max = right;
        }
        
        //如果最大值在左右结点上，交换，为了防止交换后以max为父节点的也是最大堆，所以还要调整一次
        if (max != i) {
            int temp = a[i];
            a[i] = a[max];
            a[max] = temp;
            heapAdjust(a, max, n);
        }
    }
}

void heapSort(int *a, int n) {
    //建立堆
    for (int i = n/2; i >= 1; i--) {
        heapAdjust(a, i, n);
    }
    
    for (int i = n-1; i > 0; i--) {
        int temp = a[0];
        a[0] = a[i];
        a[i] = temp;
        //由于之前堆已经建立过了，交换了堆顶后再调整一次就可以了，子节点会通过递归全部调整好
        heapAdjust(a, 0, i);
    }
}
```

