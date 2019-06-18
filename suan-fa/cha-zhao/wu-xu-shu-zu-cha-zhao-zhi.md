## 1 直接查找
时间复杂度 O(n)

## 2 使用二分法
我们借鉴快排的方法，选取一个mid值，将比A[mid]小的放在mid左边，比A[mid]大的放在右边，这样我们比较查找值target和A[mid]的大小，小就在mid左边找，大就在mid右边找，这样把搜索区间缩短一半。

然后在新的区间内继续使用上述办法，确定一个mid值，将比A[mid]小的放在左边,比A[mid]大的放在右边，然后A[mid]再和target比，这样区间又缩短一半。所以时间复杂度就是n+n/2+n/4+… = 2n，我们即可以在O(2n)的时间复杂度中完成无序数组中数据的查找，同时可知道这个target在数组中的排位。

这里出现一个问题，其实查找算法最低效率就是遍历，**时间复杂度O(n)，刚刚那个O(2n)的算法有什么意义。上面那个O(2n)的算法试用场景在于，如果我想知道某一个值在一个数组中排名多少位，那么这个时候就是这个算法的有用地方**

```c
#include<iostream>
using namespace std;
#define MAXV 1000001
int boat[MAXV];
int main(){
    int N,K;
    while(cin>>N>>K){
        int len = N;
        int i = 1;
        while(i <= N){
            cin>>boat[i];
            i++;
        }
        int right = N,left = 1;
        int start = 1,endof = N;
        bool flag = false;
        while(start <= endof){
            left = start;
            right = endof;
            int po = boat[left];
            while(left < right){
                while(left < right && boat[right] >= po)
                    right--;
                boat[left] = boat[right];
                while(left < right && boat[left] <= po)
                    left++;
                boat[right] = boat[left];
            }
                        boat[left] = po;
            if(boat[left] == K){
                flag = true;
                cout<<left<<endl;
                break;
            }
            else if(K < boat[left]){
                endof = left-1;
            }
            else{
                start = left+1;
            }
        }
        if(flag == false)
            cout<<-1<<endl;
    }       
    system("pause");
    return 0;
}

```
