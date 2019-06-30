### 题目描述

给定一个**偶数**长度的数组，其中不同的数字代表着不同种类的糖果，每一个数字代表一个糖果。你需要把这些糖果**平均**分给一个弟弟和一个妹妹。返回妹妹可以获得的最大糖果的种类数。

### 题目解析

总共有 n 个糖，平均分给两个人，每人得到  n／2 块糖，那么能拿到的最大的糖的种类数也就是 n／2 种，不可能更多，只可能更少。

所以只需要统计出糖的种类数，如果糖的种类数小于 n／2，说明拿不到 n／2种糖，最多能拿到的种类数就是当前糖的总种类数。最后，看（数量的一半）和（所有的种类）哪个先达到，取两者中较小的值即可。

举个例子：

* 极端情况1：所有糖都不重样，这种情况妹妹也只能拿到一半的糖果。

* 极端情况2：只有一种糖，这种情况妹妹只能得到一种糖果。

* 平均情况：每个糖都有两个，这种情况妹妹可以拿到所有种类，数量与**极端情况1**一样。

求糖果的种类数使用 hash 即可。

```
class Solution {
    public int distributeCandies(int[] candies) {
        Set<Integer> nums = new HashSet<>();
        for (int i = 0; i < candies.length; i++) {
            nums.add(candies[i]);
        }
        int numNums = nums.size();
        int numTarget = candies.length / 2;
        return numNums >= numTarget ? numTarget : numNums;
    }
}
```



