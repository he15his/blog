# 几道 BAT 算法面试中经常问的「字符串」问题

String 作为最常见的编程语言类型之一，在算法面试中出现的频率极高。

## 1. 验证回文串

题目来源于 LeetCode 第 125 号问题：验证回文串。这道题目是 **初级程序员** 在面试的时候经常遇到的一道算法题，而且面试官喜欢面试者手写！

### 题目描述

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

**说明：**本题中，我们将空字符串定义为有效的回文串。

**示例 1:**

```
输入: "A man, a plan, a canal: Panama"
输出: true
```

**示例 2:**

```
输入: "race a car"
输出: false
```

### 题目解析

先理解一个概念：所谓回文，就是一个正读和反读都一样的字符串。

先假设是验证一个单词 `level` 是否是回文字符串，通过概念涉及到 正 与 反 ，那么很容易想到使用双指针，从字符的开头和结尾处开始遍历整个字符串，相同则继续向前寻找，不同则直接返回 false。

而这里与单独验证一个单词是否是回文字符串有所区别的是加入了 空格 与 非字母数字的字符，但实际上的做法一样的：

一开始先建立两个指针，left 和 right , 让它们分别从字符的开头和结尾处开始遍历整个字符串。

如果遇到非字母数字的字符就跳过，继续往下找，直到找到下一个字母数字或者结束遍历，如果遇到大写字母，就将其转为小写。

当左右指针都找到字母数字时，可以进行比较的时候，比较这两个字符，如果相等，则两个指针向它们的前进方向挪动，然后继续比较下面两个分别找到的字母数字，若不相等，直接返回 false。

### 动画描述

![动画描述](https://mmbiz.qpic.cn/mmbiz_gif/D67peceibeIRgFibeYPUbiaiboBSg7eqZPickia4FYib9QFaHt0Ml5zyc9oTo9lrUTicaINwBgq19GFYeABk5gp0S2iciaTQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)动画描述

### 代码实现

注：`isLetterOrDigit` 方法确定指定的字符是否为字母或数字。

```
class Solution {
    public boolean isPalindrome(String s) {
        if(s.length() == 0)
             return true;
        int l = 0, r = s.length() - 1;
        while(l < r){
            //确定指定的字符是否为字母或数字
            if(!Character.isLetterOrDigit(s.charAt(l))){
                l++;
            }else if(!Character.isLetterOrDigit(s.charAt(r))){
                r--;
            }else{
                if(Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r)))
                    return false;
                l++;
                r--;
            } 
        }
        return true;
    }
}
```

## 2. 分割回文串

题目来源于 LeetCode 第 131 号问题：分割回文串。

### 题目描述

给定一个字符串 *s*，将 *s* 分割成一些子串，使每个子串都是回文串。

返回 *s* 所有可能的分割方案。

**示例:**

```
输入: "aab"
输出:
[
  ["aa","b"],
  ["a","a","b"]
]
```

### 题目解析

首先，对于一个字符串的分割，肯定需要将所有分割情况都遍历完毕才能判断是不是回文数。不能因为 **abba** 是回文串，就认为它的所有子串都是回文的。

既然需要将所有的分割方法都找出来，那么肯定需要用到DFS（深度优先搜索）或者BFS（广度优先搜索）。

在分割的过程中对于每一个字符串而言都可以分为两部分：左边一个回文串加右边一个子串，比如 "abc" 可分为 "a" + "bc" 。 然后对"bc"分割仍然是同样的方法，分为"b"+"c"。

在处理的时候去优先寻找更短的回文串，然后回溯找稍微长一些的回文串分割方法，不断回溯，分割，直到找到所有的分割方法。

举个🌰：分割"aac"。

1. 分割为 a + ac
2. 分割为 a + a + c，分割后，得到一组结果，再回溯到  a + ac
3. a + ac 中 ac 不是回文串，继续回溯，回溯到 aac
4. 分割为稍长的回文串，分割为 aa + c 分割完成得到一组结果，再回溯到 aac
5. aac 不是回文串，搜索结束

### 动画描述

![动画描述](https://mmbiz.qpic.cn/mmbiz_gif/D67peceibeIRgFibeYPUbiaiboBSg7eqZPick5HJRhRtzLNwQibsicubqw4UXyHVRHdwSBAEDswUQ8PtQrj77B9GCSuCw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)动画描述

### 代码实现

```
class Solution {
    List<List<String>> res = new ArrayList<>();

    public List<List<String>> partition(String s) {
        if(s==null||s.length()==0)
            return res;
        dfs(s,new ArrayList<String>(),0);
        return res;
    }

    public void dfs(String s,List<String> remain,int left){
        if(left==s.length()){  //判断终止条件
            res.add(new ArrayList<String>(remain));  //添加到结果中
            return;
        }
        for(int right=left;right<s.length();right++){  //从left开始，依次判断left->right是不是回文串
            if(isPalindroom(s,left,right)){  //判断是否是回文串
                remain.add(s.substring(left,right+1));   //添加到当前回文串到list中
                dfs(s,remain,right+1);  //从right+1开始继续递归，寻找回文串
                remain.remove(remain.size()-1);  //回溯，从而寻找更长的回文串
            }
        }
    }
    /**
    * 判断是否是回文串
    */
    public boolean isPalindroom(String s,int left,int right){
        while(left<right&&s.charAt(left)==s.charAt(right)){
            left++;
            right--;
        }
        return left>=right;
    }
}
```

## 3. 单词拆分

题目来源于 LeetCode 第 139 号问题：单词拆分。

### 题目描述

给定一个**非空**字符串 *s* 和一个包含**非空**单词列表的字典 *wordDict*，判定 *s* 是否可以被空格拆分为一个或多个在字典中出现的单词。

**说明：**

- 拆分时可以重复使用字典中的单词。
- 你可以假设字典中没有重复的单词。

### 题目解析

与上面的第二题 **分割回文串** 有些类似，都是拆分，但是如果此题采取 深度优先搜索 的方法来解决的话，答案是超时的，不信的同学可以试一下~

为什么会超时呢？

因为使用 深度优先搜索 会重复的计算了有些位的可拆分情况，这种情况的优化肯定是需要 动态规划 来处理的。

如果不知道动态规划的，可以看一下小吴之前的[万字长文，比较详细的介绍了动态规划的概念。](http://mp.weixin.qq.com/s?__biz=MzUyNjQxNjYyMg==&mid=2247484350&idx=1&sn=fc88aa125f5a5269575b4c4d83774f41&chksm=fa0e6c3fcd79e5297257a05b8c75898b4059b1193956c702ff5ef3f2d8d46432bb7484bf6428&scene=21#wechat_redirect)

在这里，只需要去定义一个数组 boolean[] memo，其中第 i 位 memo[i] 表示待拆分字符串从第 0 位到第 i-1 位是否可以被成功地拆分。

然后分别计算每一位是否可以被成功地拆分。

### 代码实现

```
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        int n = s.length();
        int max_length=0;
        for(String temp:wordDict){
            max_length = temp.length() > max_length ? temp.length() : max_length;
        }
        // memo[i] 表示 s 中以 i - 1 结尾的字符串是否可被 wordDict 拆分
        boolean[] memo = new boolean[n + 1];
        memo[0] = true;
        for (int i = 1; i <= n; i++) {
            for (int j = i-1; j >= 0 && max_length >= i - j; j--) {
                if (memo[j] && wordDict.contains(s.substring(j, i))) {
                    memo[i] = true;
                    break;
                }
            }
        }
        return memo[n];
    }
}
```

## 4. 反转字符串

题目来源于 LeetCode 第 344 号问题：反转字符串。面试官最喜欢让你手写的一道算法题！

### 题目描述

编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 `char[]` 的形式给出。

不要给另外的数组分配额外的空间，你必须**原地修改输入数组**、使用 O(1) 的额外空间解决这一问题。

你可以假设数组中的所有字符都是 ASCII 码表中的可打印字符。

**示例 1：**

```
输入：["h","e","l","l","o"]
输出：["o","l","l","e","h"]
```

**示例 2：**

```
输入：["H","a","n","n","a","h"]
输出：["h","a","n","n","a","H"]
```

### 题目解析

这道题没什么难度，直接从两头往中间走，同时交换两边的字符。注意需要白板编程写出来即可，也注意千万别回答一句使用 reverse() 这种高级函数来解决。。。

### 动画描述

![动画](https://mmbiz.qpic.cn/mmbiz_gif/D67peceibeIRgFibeYPUbiaiboBSg7eqZPickKHMSV2FUELMM1IiamePkicJHJsWDjvTMGXHBGJ2NDjy5icK5y4gRVtASw/640?wx_fmt=gif&wxfrom=5&wx_lazy=1)动画

### 代码实现

```
class Solution {
public:
    string reverseString(string s) {
        int i = 0, j = s.size() - 1;
        while (i < j){
            swap(s[i],s[j]);
            i++;
            j--;
        }
        return s;
    }
};
```

## 5. 把字符串转换成整数

题目来源于剑指 offer 。

### 题目描述

将一个字符串转换成一个整数，字符串不是一个合法的数值则返回 0，要求不能使用字符串转换整数的库函数。

### 题目解析

这道题要考虑全面，对异常值要做出处理。

对于这个题目，需要注意的要点有：

- 指针是否为空指针以及字符串是否为空字符串；
- 字符串对于正负号的处理；
- 输入值是否为合法值，即小于等于'9'，大于等于'0'；
- int为32位，需要判断是否溢出；
- 使用错误标志，区分合法值0和非法值0。

### 代码实现

```
public class Solution {
    public int StrToInt(String str) {
    if (str == null || str.length() == 0)
        return 0;
    boolean isNegative = str.charAt(0) == '-';
    int ret = 0;
    for (int i = 0; i < str.length(); i++) {
        char c = str.charAt(i);
        if (i == 0 && (c == '+' || c == '-'))  /* 符号判定 */
            continue;
        if (c < '0' || c > '9')                /* 非法输入 */
            return 0;
        ret = ret * 10 + (c - '0');
    }
    return isNegative ? -ret : ret;
  }
}
```