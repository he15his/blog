直接从表第一个开始，顺序查到最后一个，时间复杂度 O(n)

```c
int seq_search(int array[], int n, int key)
{
    int i;
    for(i = 0; i < n; i++)
    {
        if(key == array[i])
        {
            return i;   //查找成功
        }   
    }
    return -1;          //查找失败
}
```

