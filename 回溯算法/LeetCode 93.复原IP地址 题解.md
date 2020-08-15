## [93. 复原IP地址](https://leetcode-cn.com/problems/restore-ip-addresses/)

### 解法一  暴力

#### 思路

三重循环暴力求解，将字符串划分为四段。

####  实现细节

三重循环将字符串分为四段，判断分割合法性，若合法则加入结果数组res。



####  代码

```cpp
class Solution {
public:
    bool Isvalid(vector<string> str) {	// 判断分割是否满足条件
        for(int i = 0; i < 4; i++) {
            if(str[i].size() > 1 && str[i][0] == '0') { // 含有前导0报错
                return false;
            } else if(stoi(str[i]) > 255) {	//分段大于255报错
                return false;
            }
        }
        return true;
    }
    
    vector<string> restoreIpAddresses(string s) {
        int len = s.size();
        vector<string> res; //存储结果
        for(int i = 1; i < 4; i++) {
            for(int j = 1; j < 4; j++) {
                for(int k = 1; k < 4; k++) {
                    int m = len - i - j - k;
                    if(m > 0 && m < 4) {  // 保证分割位数合法
                        vector<string> str;	// 存储本次分割得到的四个子字符串
                        str.push_back(s.substr(0, i));
                        str.push_back(s.substr(i, j));
                        str.push_back(s.substr(i + j, k));
                        str.push_back(s.substr(i + j + k, m));
                        if(Isvalid(str)) {
                            string pos = "";
                            for(int i = 0; i < 4; i++) {
                                pos += str[i];
                                if(i != 3) {
                                    pos += ".";
                                }
                            }
                            res.push_back(pos);
                        }
                    }
                }
            }
        }
        return res;
    }
};
```

####  复杂度

设字符串长度为N，三重循环，时间复杂度为O(3^3 = 27)，每次循环需要处理一遍字符串，复杂度为O(N)，所以总时间复杂度为O(N)；

维护了记录分割结果的数组，空间复杂度O(N)。



### 解法二  DFS回溯

#### 回溯算法：

**回溯算法的核心思想**：**能进则进，进不了就换，换不了就退** 。

**回溯算法的一般套路**

1.定义问题的解空间。 

2.在搜索的过程中求解。

3.显式约束用于剪枝，隐含约束用于求解最值，可以辅助剪枝。

4.回溯算法的一个重要特点是在搜索的过程中，同时产生解空间。在搜素的任何时刻，仅仅保留从开始结点到当前结点的路径。



#### 思路：

**题意理解：题目的意思就是要把一个字符串进行划分，划分为4段，每段之间用点分隔，每段的数据大小保证在0~255**。**为了把一个字符串划分成四段，我们只要找出所有满足条件的加点位置（每一种方案 包含3个点的位置）就行。**

尝试所有的可能，每找到一种加点位置的方案，如果合理，那就遍历整个字符串，根据加点位置构造出相应的结果。

使用DFS回溯时，对于每一次划分可以选择为这一段划分1，2，3个字符，然后继续划分下一段，一共划分成四段，**所以本题的解空间就是一个三叉树，每一次划分可以选择划分1，2，3个字符**。问题求解的过程就是遍历这棵三叉树的过程，在遍历的过程中求解。

**为了加快搜索，尝试所有可能时，要考虑一些约束，从而剪枝（加快搜索），本题中的划分为4段，每段的数据大小0~255就是约束，还有就是划分的每一段不能含有前导0，所以划分时要考虑到下一段的头部字符，比如“20019211135”划分时候 第一段只能为200，不能为2，20，否则会使得第二段有前导0**

**可以通过画一个图来理解搜索和回溯的过程**

举个例子：“25525511135”，引用网上的一个图， **强烈建议自己画一个要搜索的树理解搜索过程**。



####  实现细节

用一个字符串数组res来记录所有的有效划分结果，用一个数组pointpos来记录加点的位置,  构造相应的方案，加入res中。

**更多详细解释见代码注释。**



#### 代码

```cpp
class Solution {
private:
    vector<string>res; //记录所有的可能情况
    vector<int>pointpos; //记录加点的位置
    
    //通过记录的加点位置，构造出相应地划分方案
    //对字符串s, 根据加点位置数组pointpos, 对字符串s进行加点划分, 结果存在tempstr
    void splitIpAddress(string &s, string &tempstr, vector<int>&pointpos)
    {
        int begin = 0; //记录加点位置数组的下标
        for (int i = 0; i < s.size(); ) //遍历字符串s
        {
            //i为加点位置 加点
            if (begin < pointpos.size() &&  i == pointpos[begin])
            {
                tempstr.push_back('.');
                begin++;
            }
            //添加字符串s的对应字符
            else
            {
                tempstr.push_back(s[i]);
                i++;
            }
        }
        res.push_back(tempstr); //划分结果加入res
    }

    //搜索与回溯 
    // s为要划分的字符串， depth代表深度（划分的段数），index表示剩余要划分部分的开始下标,[index, s.size() - 1]继续划分
    void backtrack(string &s, int depth, int index)
    {
        if(depth == 4) //深度为4的话 表示已经加了4个点，最后一个点无实际意义
        {
            if (index == s.size()) //后面没有多余的字符串可以划分， 合理划分
            {
                string tempstr; //记录划分结果
                splitIpAddress(s, tempstr, pointpos); //构造划分方案
            }
            return;
        }
        else
        {
            int count = 0; //记录该段的数据大小
            for (int i = index; i < index + 3 && i < s.size(); i++) //最多能把三位划分为一段（每段0~255）
            {
                count = count * 10 + (s[i] - '0'); //计算划分的这一段的数据结果
                if (count < 256)// 确保是合理的划分 0~255
                {
                    int digit = i + 1 - index; //划分段的位数
                    if (digit == 2) //一段中有两位时  比如“20019211135”划分时候 第一段只能为200
                    {
                        if (s[i +  1 - digit] == '0') //有前导0 前导0也是一个约束条件
                        {
                            break;	//结束当前段的划分
                        }
                    }
                    pointpos.push_back(i + 1); // 选择加点位置为i + 1
                    backtrack(s, depth + 1, i + 1); //递归划分剩余部分, [i + 1, s.size())部分继续划分
                    pointpos.pop_back(); // 去除选择的加点位置，重新选择加点位置
                }
            }
        }
    }
public:
    vector<string> restoreIpAddresses(string s) {
        backtrack(s, 0, 0); //搜索回溯
        return res;
    }
};
```



#### 复杂度分析

时间复杂度：这是遍历一棵3叉树，树的深度为4，每次找到一个加点位置的方案，构造出一种划分结果需要遍历的字符串，所以递归的时间复杂度为O(3^4 = 81)，每次要处理一遍字符串，时间复杂度为O(N)，所以总时间复杂度为O(81N) = O(N)

空间复杂度：考虑记录划分方法的数组，使用一个数组来记录加点的位置，空间复杂度为O(N)



#### Tips:

对本题进行一定的扩展：假设给定长度为N的字符串，要求分为K个整数，每个整数的位数不超过M，要求输出全部可能解，此时我们给出以上两种方法的时间复杂度。

对于暴力求解法，实际上要进行K - 1重循环，每重循环的指标范围为 1 到 M，判断合理需要处理长度为N的字符串，所以总时间复杂度为：
$$
O(M^{K - 1} * N)
$$
而对于DFS回溯算法，相当于遍历一棵M叉树（因为整数位数从1到M），且树的深度为K，且每次仍然要处理一遍长度为N的字符串，所以总时间复杂度为：
$$
O(M^K * N)
$$
