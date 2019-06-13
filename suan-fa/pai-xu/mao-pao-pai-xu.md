冒泡（Bubble）排序——相邻交换



![img](mao-pao-pai-xu.assets/20160316103848750-20190613203028196.jpeg)

```c
//冒泡排序
void bubbleSort(int *a, int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (a[j] < a[j+1]) {
                int temp = a[j];
                a[j] = a[j+1];
                a[j+1] = temp;
            }
        }
    }
}
```

