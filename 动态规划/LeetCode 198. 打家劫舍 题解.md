## [LeetCode 198. 打家劫舍 题解](https://leetcode-cn.com/problems/house-robber/)

![image-20200812151124526](C:\Users\lzh222333\AppData\Roaming\Typora\typora-user-images\image-20200812151124526.png)

### 解法1：直观的动态规划

#### 思路：

动态规划常常用来解决具有最优子结构的问题，即每次仅试图解决每个子问题，可以通过记忆化存储，以便下次需要同一个子问题解的时候直接查表。可以理解动规有四个要素：状态表示、状态转移方程、初始状态、答案表示。

在本题中我们使用一个dp数组来表示遍历到每个房屋在偷与不偷的情况下所能获取的最高金额，其中dp\[i][0]为在i个房屋不进行偷窃，dp\[i][1]为在i个房屋进行偷窃，此为状态表示；

在第i个房屋时，若选择偷窃，则不能偷窃第i-1个房屋，偷窃金额为经过第i-1个房屋的不偷窃的最高金额加上当前房屋的金额；若选择不偷窃，则在第i-1个房屋既可以偷也可以不偷，则在第i-1个房屋的这两种状态金额的更大值即为不偷窃第i个房屋的最高金额，由此，状态转移方程为：
$$
\left\{ \begin{array}{l}
	dp\left[ i \right] \left[ 0 \right] =\max \left( dp\left[ i-1 \right] \left[ 0 \right] ,dp\left[ i-1 \right] \left[ 1 \right] \right)\\
	dp\left[ i \right] \left[ 1 \right] =dp\left[ i-1 \right] \left[ 0 \right] +nums\left[ i \right]\\
\end{array} \right.
$$


对于初始状态，当i=0时，i-1超出了数组范围，无法使用状态转移方程，因此单独分析，当i=0时，偷即为nums[0]，不偷即为0；

对于答案表示，我们需要走完所有房屋获取最大金额，即dp\[n-1][0]和dp\[n-1][1]的更大值，n为房屋总数。

#### 实现细节：

首先有个好习惯，判断数据是否为空，然后建立全0的dp数组，并开始遍历，若i=0记得给初始状态，然后按照状态转移方程编写代码，最后返回max(dp\[n-1][0],dp\[n-1][1])。

#### Python3代码：

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        n = len(nums)
        if n==0:
            return 0
        # 创建dp数组，dp[i][0]为不偷这家所能达到的最大值，dp[i][1]为偷这家所能达到的最大值
        dp = [[0,0] for i in range(n)]
        for i in range(n):
            # 初始条件
            if i==0:
                dp[i][0] = 0
                dp[i][1] = nums[i]
            # 动规方程
            else:
                # 这家不偷，则上家可以偷
                dp[i][0] = max(dp[i-1][0],dp[i-1][1])
                # 这家偷，则上家不能偷，但是可以加上这家的价值
                dp[i][1] = dp[i-1][0] + nums[i]
        return max(dp[n-1][0],dp[n-1][1])
```

#### 复杂度分析：

时间复杂度：O(n)，每间房屋遍历一遍；

空间复杂度：O(n)，房屋长度的dp数组用于记录子问题的解。



### 解法2：动态规划优化+滚动数组

#### 思路：

我们可以不需要记录每次是否进行了偷窃，直接将两者中的更大值作为dp数组的值，这样若偷第i个房屋，则不能偷第i-1个房屋，只能使用第i-2个房屋的最大值加上当前房屋的金额；若不偷，则保持为第i-1个房屋的金额，状态转移方程为：
$$
dp\left[ i \right] =\max \left( dp\left[ i-2 \right] +nums\left[ i \right] ,dp\left[ i-1 \right] \right) 
$$


边界条件为：
$$
\left\{ \begin{array}{l}
	dp\left[ 0 \right] =nums\left[ 0 \right]\\
	dp\left[ 1 \right] =\max \left( nums\left[ 0 \right] ,nums\left[ 1 \right] \right)\\
\end{array} \right. 
$$


最终答案为dp[n-1]，n为房屋数量。

同时dp数组我们可以不显式地创建，因为第i间房屋的时候我们只需要第i-1间和第i-2间的数据，由此可以仅通过两个变量滚动进行更新，以达到节省空间的目的。

#### Python3代码：

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        n = len(nums)
        if n==0:
            return 0
        if n==1:
            return nums[0]
        first, second = nums[0], max(nums[0],nums[1])
        for i in range(2,n):
            # 动规方程
            first, second = second, max(second,first+nums[i])
        return second
```

#### 复杂度分析：

时间复杂度：O(n)

空间复杂度：O(1)，仅额外使用两个变量