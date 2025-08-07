---
title: 从最长公共子序列理解如何从二维数组压缩为一维数组
date: 2025-08-07 17:34:30
tags: [算法笔记,动态规划]
---
`题目：给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0 。`
`一个字符串的 子序列 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。`
`例如，"ace" 是 "abcde" 的子序列，但 "aec" 不是 "abcde" 的子序列。`
`两个字符串的 公共子序列 是这两个字符串所共同拥有的子序列。`
`示例 1：`
`输入：text1 = "abcde", text2 = "ace" `
`输出：3 `
`解释：最长公共子序列是 "ace" ，它的长度为 3 。`

<!-- more -->

## dp数组含义

本题dp数组表示两个字符串分别的前i个字符和前j个字符的状态（最长公共子序列的长度）而不包含第i和j个字符本身。因为这样方便后续直接从1开始遍历操作dp[i-1] [j-1]而不需要考虑数组越界。

## 核心逻辑：递推公式

使用for循环两层遍历，内层中首先判断第i-1和j-1个字符是否相等（因为i、j是从1开始遍历，因此必须减一开始遍历才能遍历完全）若相等则直接等于dp[i-1] [j-1]+1.若不相等，则取两个字符串分别减一之后的最大值。

```
if(chars1[i-1]==chars2[j-1]){
dp[i][j]=dp[i-1][j-1]+1;
}else{
dp[i][j]=Math.max(dp[i-1][j],dp[i][j-1]);
}
```

## 遍历顺序

遍历如图：是从上到下、从左到右依次遍历。每次状态取决于左上、上、左的状态。

|      | ""   | a    | c    | e    |
| ---- | ---- | ---- | ---- | ---- |
| ""   | 0    | 0    | 0    | 0    |
| a    | 0    | 1    | 1    | 1    |
| b    | 0    | 1    | 1    | 1    |
| c    | 0    | 1    | 2    | 2    |
| d    | 0    | 1    | 2    | 2    |
| e    | 0    | 1    | 2    | 3    |

由图可知，一个位置的元素是由其上、左上、左三个元素状态推出来的

初始化时将所有元素置为0.

二维数组解法：

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        char[] chars1=text1.toCharArray();
        char[] chars2=text2.toCharArray();
        int[][] dp=new int[text1.length()+1][text2.length()+1];
        for(int[] rows:dp){
            Arrays.fill(rows,0);
        }
        for(int i=1;i<=chars1.length;i++){
            for(int j=1;j<=chars2.length;j++){
                if(chars1[i-1]==chars2[j-1]){
                    dp[i][j]=dp[i-1][j-1]+1;
                }else{
                      dp[i][j]=Math.max(dp[i-1][j],dp[i][j-1]);
                }
            }
        }
        return dp[text1.length()][text2.length()];
    }
}
                
```



## 一维数组解法

由上述二维数组解法可知：dp[i] [j] 依赖于三个方向

而一维数组正是记录这三个方向的变量来代替二维时i的数组，从而变为了一维数组，并在每次i更新时重新赋值

一维数组时需要维护的变量：

- dp[i-1] [j-1] -> 用变量 pre 来保存
- dp[i-1] [j]   -> 就是当前 dp[j]
- dp[i] [j-1]   -> 当前行内，已经更新过的 dp[j-1]

这里首先贴出一维数组代码实现：

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int n1=text1.length();
        int n2=text2.length();
        char[] chars1=text1.toCharArray();
        char[] chars2=text2.toCharArray();
        int[] dp=new int[n2+1];
        Arrays.fill(dp,0);
        for(int i=1;i<=n1;i++){
            int pre=0;
            for(int j=1;j<=n2;j++){
                int temp=dp[j];
                if(chars1[i-1]==chars2[j-1]){
                    dp[j]=pre+1;
                }else{
                    dp[j]=Math.max(dp[j],dp[j-1]);
                }
                pre=temp;
            }
        }
        return dp[n2];
    }
}
```

细节：在每一行开始（每个 i 的循环开始）时，还没进入 j 的时候，其实就相当于在二维 dp 里 j=0 的那一列【而一维数组与二维数组dp含义大致相同，都是字符串第i-1个字符及以前的最长公共子序列】

而 dp[i-1] [0] == 0（空字符串和任意字符串的LCS都是0），这时候 dp[i-1] [j-1] 也还没出现。

![image-20250807174907533](E:\blog\iamhash.github.io\source\_posts\my-lcs.assets\image-20250807174907533.png)

可得，每一行一开始，pre 其实对应的是 dp[i-1] [0]，它是 0。

当你往右走的时候（j从1到n），pre 就一步步记录了上一轮的左上角值（dp[i-1] [j-1]）

以下是一维数组遍历时各个变量的表示：

| 变量    | 表示什么                       | 注意点                       |
| ------- | ------------------------------ | ---------------------------- |
| dp[j]   | 当前行的 dp[i] [j]             | 但它马上就会被覆盖，需要暂存 |
| dp[j-1] | 当前行的 dp[i] [j-1]           | 已经更新过                   |
| pre     | 上一行的 dp[i-1] [j-1]         | 要靠 temp 来更新             |
| temp    | 保存旧的 dp[j]，即 dp[i-1] [j] | 每次用于下一轮更新 pre       |

内层循环还是对第二个字符串的遍历

因此，从二维变成一维的本质，是在**模拟二维中 dp[i] [j] 所依赖的三个方向的值（上、左、左上），然后通过额外变量（如 pre, temp）来维护这些值的位置关系，逻辑保持不变。**