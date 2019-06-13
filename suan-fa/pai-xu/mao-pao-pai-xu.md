冒泡（Bubble）排序——相邻交换



![image-20190613205952434](mao-pao-pai-xu.assets/image-20190613205952434.png)

```c
//冒泡排序(从小到大)
void bubbleSort(int *a, int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (a[j] > a[j+1]) {
                int temp = a[j];
                a[j] = a[j+1];
                a[j+1] = temp;
            }
        }
    }
}
```

