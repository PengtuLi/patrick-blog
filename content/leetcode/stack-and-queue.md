## 理论基础

:::info[TLDR]

- 栈：先进后出
- 队列：先进先出

:::

`C++`：栈和队列属于STL，但往往将其认为是container adapter，因为其底层实现可以
是vector，list，deque, etc.

`python`

- python里list与栈类似
- deque可以模拟队列也可以模拟栈
- `queue.Queue`不可以获取队头元素且不弹出，不推荐
- 均为可迭代对象，如果想要`deque->char`怎么办呢？可以join

:::tip

可以用双指针+list替代栈，如果不让用栈的话

:::

## [LC232-用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks/description/)

:::info[TLDR]

用栈实现队列的操作，如push, pop, peek, empty, etc.

:::

解法：

- 双栈模拟队列，很简单，$O(n)$ 复杂度，需要注意的是
  - 何时进行input->output的迁移
  - output为空时的处理

## [LC225-用队列实现栈](https://leetcode.cn/problems/implement-stack-using-queues/)

:::info[TLDR]

队列实现栈的操作，如push, pop, top, empty
类似`LC225`
:::

解法：

- 双队列模拟栈
  - 插入时队尾插入，弹出时需要弹出队尾的元素，因此弹出$n-1$个元素，最后一个就是要输出的
  - 复杂度插入$O(1)$，弹出$O(n)$,平均复杂度$O(n^2)$
- 单队列
  - 复杂度，插入$O(n)$,平均复杂度$O(n^2)$
  - 关键是让每次插入后队头都是栈头元素，因此关键就是push的时候先插入元素，再把里面的元素都弹出来再插入

## [LC20-括号匹配](https://leetcode.cn/problems/valid-parentheses/)

:::info[TLDR]

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

一个有效的例子：`{[]}{}()`

:::

解法：用栈做匹配，基本的思路就是是左边的符号就推入栈，遇到右边的符号是就弹出栈进行匹配。
关键是确定何时匹配不上,`$`代表匹配成功的部分

- `[$`or`$[`左边符号多了，对应的是栈不为空
- `]$`or`$]`，同理，需要匹配时栈为空
- 不匹配

<details>

<summary>python</summary>

```python showLineNumbers
from collections import deque

def advers_char(c):
    d={
        '}':'{',
        ']':'[',
        ')':'('
    }
    return d[c]

class Solution:
    def isValid(self, s: str) -> bool:
        l = deque()
        for c in s:
            if c == '}' or c == ']' or c == ')':
                advers=advers_char(c)
                # case 1: right
                if len(l)==0:
                    return False
                top=l.pop()
                if top==advers:
                    continue
                else:
                    # case 3: not match
                    return False
            l.append(c)
        # case 2: left
        if len(l)!=0:
            return False
        return True
```

</details>

## [LC1047-删除字符串中的所有相邻重复项](https://leetcode.cn/problems/remove-all-adjacent-duplicates-in-string/)

:::info[TLDR]

不停删除字符串内的两个相邻且相同的字符，返回删除后的结果

:::

解法：利用栈，思考何时要弹出（top与next相同的时候），删除后出现新的对子怎么办？
只可能top+next,所以重复一样的操作就好了，复杂度$O(n)$

## [LC150-逆波兰表达式求值](https://leetcode.cn/problems/evaluate-reverse-polish-notation/)

:::info[TLDR]

根据逆波兰表示法，求表达式的值。
所谓逆波兰，就是指运算符在两个操作数后面,我们日常使用的一般是中缀，如 (3+5)\*8,
需要加括号确定优先级，而后缀表达式的特点就是不需要加括号。

逆波兰表示法实际上是一种后序遍历（Post-order traversal）：

- 对于表达式树，后序遍历的顺序就是后缀表达式

对于表达式树（特殊的二叉树），_别的树只靠前序或后序无法还原哦_：

- 后序遍历实际上可以唯一还原树结构！
- 前序遍历也可以唯一还原树结构！
- 中序遍历会丢失树的结构信息

当我们对表达式树进行中序遍历时，得到的结果是标准的中缀表达式，
但这个表达式**可能无法**唯一还原原来的树结构。

```text
表达式树：   +
           /   \
          1     *
               / \
              2   3

中序遍历：1 + 2 * 3
```

:::

解法：

- 很简单，就是遍历表达式,复杂度$O(n)$，用栈存储数，遇到运算符弹出操作数并压入结果。
- 注意字符串与整数的变换；注意整除向零截断可以用绝对值；
