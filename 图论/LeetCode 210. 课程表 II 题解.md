## [210. 课程表 II](https://leetcode-cn.com/problems/course-schedule-ii/)

#### **题目分析：**

​    题目给出了课程之间的先修关系，要求我们找出修完所有课程的正确顺序。首先，我们可以将这道题抽象为图的形式，其中先修关系为图中的边，每个课程为图中的点。另外，从题目的实际意义出发，这些课程之间是不存在循环先决条件的，也就是说不会出现几个课程的先修关系构成一个环，导致这些课程无法完成，所以如果这道题有解那么这个图一定是无环图。综上，有向图+无环&有环判断+先后顺序=拓扑排序，这是一道拓扑排序的模板题！

 

#### **思路&实现：**

**拓扑排序：**

将一个有向无环图中所有顶点在不违反先决条件关系的前提下，排成线性序列的过程称为拓扑排序。线性序列称作一个拓扑序列。

（1）   将所有入度为0的点入队；

（2）   输出一个入度为0的点；

（3）   将与该弧头指向的所有点的入度-1，若入度减至0，则将此点入队，重复（2）和（3）直至队空。

在本题中，拓扑排序中的（1）对应找到所有没有先修课程的课程，先将这些课程入队等待处理，（2）对应当找到一个可修课程后，（3）对应将所有以这门为先修课程的课程标记，并在这里找出已经完成所有先修课程的课程，将其入队。即队列中仅保存所有先修课程以完成的课程。



**前向星&链式前向星：**

​    本公众号中有关图论的题解一般采用链式前向星方式存图。这里简要介绍一下：

前向星是以存储边的方式来存储图，先将边读入并存储在连续的数组中，然后按照边的起点进行排序，这样数组中起点相等的边就能够在数组中进行连续访问了。它的优点是实现简单，容易理解，缺点是需要在所有边都读入完毕的情况下对所有边进行一次排序，带来了时间开销，实用性也较差，只适合离线算法。

链式前向星和邻接表类似，也是链式结构和线性结构的结合，每个结点i都有一个链表，链表的所有数据是从i出发的所有边的集合（对比邻接表存的是顶点集合），边的表示为一个四元组(u, v, w, next)，其中(u, v)代表该条边的有向顶点对，w代表边上的权值，next指向下一条边。具体的，我们需要一个边的结构体数组 edge[MAXM]，MAXM表示边的总数，所有边都存储在这个结构体数组中，并且用head[i]来指向 i 结点的第一条边。

```c++
int cnt=0,head[MAXN]; 
struct edges{
    int u,v,w,next;
} edge[MAXM];
void addEdge(int u, int v, int w) {
    edge[++cnt].u = u;
    edge[cnt].v = v;
    edge[cnt].w = w;
    edge[cnt].next = head[u];
    head[u] = cnt;
}
```



#### **代码：**

```c++
class Solution {
public:
    struct edge{
        int to,next;
    } e[10050];
    queue<int> q;               //用于存放入度为0的点
    vector<int> res;            //用于存放结果
    /*head和cnt为存图结构，flag表示初始入度为0的点*/
    int head[5050],cnt=0,flag[5050];    
    void add(int y,int x){
        e[++cnt].to=y;
        e[cnt].next=head[x];
        head[x]=cnt;
    }
    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        memset(head,-1,sizeof(head));
        for(int i=0;i<prerequisites.size();i++){
            add(prerequisites[i][0],prerequisites[i][1]);
            flag[prerequisites[i][0]]+=1;
        }
        //将所有入度为0的点入队，对应(1)
        for(int i=0;i<numCourses;i++) if(flag[i]==0) q.push(i);
        while(!q.empty()){
            //取到一个入度为0的点，对应(2)
            int tmp=q.front();
            q.pop();
            res.push_back(tmp);
            //将与该弧头指向的所有点进行操作，对应(3)
            for(int i=head[tmp];i!=-1;i=e[i].next){
                flag[e[i].to]-=1;
                if(!flag[e[i].to]) q.push(e[i].to);
            }
        }
        vector<int> a;
        //没有解返回空数组
        if(res.size()<numCourses) return a;
        return res;
    }
};
```

