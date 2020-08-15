## [leetcode 328. 奇偶链表](https://leetcode-cn.com/problems/odd-even-linked-list/)

### 解法1：简单遍历 + 奇偶结点分离

#### 思路：

由于题目要求使用原地算法，因此我们不必改变结点本身的数值，只需调整各结点的指针域。我们可以把所有奇数位结点链再一起形成一个奇链表，再把所有偶数位结点链在一起形成一个偶链表。又由于题目的时间复杂度要求是$O({\rm nodes})$，因此我们只能对原链表进行线性遍历，在遍历过程中交替地将当前结点接到奇链表和偶链表上。最后再把偶链表的表头接在奇链表的表尾。



#### 实现细节：

由于我们是要把原链表的结点接到奇链表和偶链表的表尾，最后还要把偶链表的表头接到奇链表的表尾，因此我们需要建立三个指针变量：奇链表表尾`r_odd`，偶链表表尾`r_even`和偶链表表头`h_even`。原链表的表头就是奇链表的表头，即`head`，我们最后返回的也是这个指针。

在遍历过程中，依次将当前结点接在对应的链表表尾后。

维护一个变量记录当前遍历到的结点是奇数位还是偶数位，在这里使用布尔型变量，占1字节，而使用int型变量占4字节。



#### 代码：

```c++
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        if(!head || !head->next) { //如果链表无结点或只有一个结点，那么直接返回原链表即可
            return head;
        }
        ListNode* r_odd = head; //奇链表的表尾
        ListNode* r_even = head->next; //偶链表的表尾
        ListNode* h_even = head->next; //偶链表的表头

        ListNode* t = head->next->next; //原链表中将要遍历到的结点
        //将奇链表和偶链表的表尾结点指针置空
        r_odd->next = NULL; 
        r_even->next = NULL;

        bool odd = true;  //维护布尔变量odd表示当前遍历到的结点编号是否为奇数
        while(t) { //逐个遍历原链表的结点
            if(odd) { //当前结点应接到奇链表上
                r_odd->next = t;
                t = t->next;
                r_odd = r_odd->next;
                r_odd->next = NULL;
                odd = false;
            } else { //当前结点应接到偶链表上
                r_even->next = t;
                t = t->next;
                r_even = r_even->next;
                r_even->next = NULL;
                odd = true;
            }
        }

        r_odd->next = h_even; //将偶链表的表头接在奇链表的表尾后
        return head;
    }
};
```

#### 复杂度：

遍历一遍链表，时间复杂度为O(N)，满足题目要求；

维护了若干个变量，空间复杂度为O(1)





### 解法2：代码优化

#### 实现细节：

解法1中，每次遍历一个结点的过程就作为一次循环的过程，因此我们需要在循环中判断此次遍历的结点是奇数位还是偶数位。但其实我们可以将遍历相邻的一个奇结点和一个偶结点的过程共同作为一次循环。这时，要将循环条件改为判断偶链表尾结点以及它在原链表中的下一个结点是否同时存在。如果这两个结点同时存在，那么就能够保证下一轮循环过程中奇链表尾指针和偶链表尾指针均能继续后移，且移动后的奇链表尾指针指向的结点一定不为空（因为最后要在奇链表表尾接偶链表）。而如果这两个结点有一个不存在，那么就可以结束循环，因为原链表已遍历完成，偶链表尾指针是指向最后一个偶结点还是指向NULL是无关紧要的。

由于每次循环中一定会更新当前奇链表尾结点和偶链表尾结点的指针域，因此无需再像解法1中那样每次将两个结点的指针域置空。



#### 代码：

```c++
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        if(!head) {
            return NULL;
        }
        ListNode *r_odd = head, *r_even = head->next, *h_even = r_even;
        while(r_even && r_even->next) {
            r_odd->next = r_even->next;
            r_odd = r_odd->next;
            r_even->next = r_odd->next;
            r_even = r_even->next;
        }
        r_odd->next = h_even;
        return head;
    }
};
```



#### 复杂度：

复杂度与解法一均相同，只不过空间复杂度上，相较于解法一减少了变量的使用。