---
weight: 1
title: "Algorithm 算法学习"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Algorithm 算法学习"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Algorithm, Note]
categories: [Algorithm算法]

lightgallery: true

toc:
  auto: false
---

算法相关，对于非科班出身的人来说，只能慢慢学习和体会了

<!-- more -->

# 算法

- 有穷性(Finiteness)
算法的有穷性是指算法必须能在执行有限个步骤之后终止；
- 确切性(Definiteness)
算法的每一步骤必须有确切的定义；
- 输入项(Input)
一个算法有0个或多个输入，以刻画运算对象的初始情况，所谓0个输入是指算法本身定出了初始条件；
- 输出项(Output)
一个算法有一个或多个输出，以反映对输入数据加工后的结果。没有输出的算法是毫无意义的；
- 可行性(Effectiveness)
算法中执行的任何计算步骤都是可以被分解为基本的可执行的操作步，即每个计算步都可以在有限时间内完成（也称之为有效性）。

## 聚合

1. 先准备一个dict
2. 循环数据
3. 准备唯一键
4. 如果唯一键没在dict中，生成新的dict2
5. 并且 `dict[key] = dict2`
6. 如果在，`dict2 = dict[key]`
7. 然后修改dict2

利用字典在原引用修改的特性，把数据进行聚合分组，每次循环都会根据唯一键从临时数据字典dict处取出要聚合到这个key对应的字典中，然后对其进行操作。

## 找出数组中，只出现一次的两个数字

list = [2, 4, 3, 6, 3, 2, 5, 5]

核心思路：

1、数组中全部数据异或操作后，依次对数组中的每个元素进行异或（相同位为0，不同为1）操作，得到0000 0010。

2、倒数第二位是1，说明我们要找的那两个只出现一次的数字，倒数第二位是不同的。(会出现不同，是因为这两个数不同，所以至少有一位是不同的)

3、下面根据每个数二进制倒数第二位是不是1来分成两组，倒数第二位为1的是{2, 3, 6, 3, 2}，倒数第二位为0的是{4, 5, 5}。

4、接下来对这两个数组分别进行异或操作，剩下的数字就是只出现一次的数字。

为什么分组可以实现：因为4，6是不同的两个数，它们二进制的至少某一位是不同的（记为N位），所以把这位是1的分在一起，试想所有的数在N位不同的是4或6中的一个，加上其它的数，其它的数都是成对的，所有其它的数也只会分成两组（其它的数N位同样只会是1或0），一组包含4和多个重复的数，一组包含6和多个重复的数，重复的数是可以消去的。

## A + B 问题

给出两个整数 aa 和 bb , 求他们的和。

- 样例
如果 a=1 并且 b=2，返回3。

- 挑战
显然你可以直接 return a + b，但是你是否可以挑战一下不这样做？（不使用++等算数运算符）

- 说明
a和b都是 32位 整数么？

    是的
- 我可以使用位运算符么？

    当然可以

思路：运用位运算模拟加法

```python

class Solution:
    """
    @param a: An integer
    @param b: An integer
    @return: The sum of a and b 
    """
    def aplusb(self, a, b):
        # write your code here
        if a == -b:
            return 0
        else:
            while b != 0:
                a, b = a ^ b, (a & b) << 1
                # 每次去算进位的地方，进位的和a一直相加
            return a

```

主要利用异或运算来完成 
- 异或运算有一个别名叫做：不进位加法
- 那么a ^ b就是a和b相加之后，该进位的地方不进位的结果
- 然后下面考虑哪些地方要进位，自然是a和b里都是1的地方
- a & b就是a和b里都是1的那些位置，a & b << 1 就是进位
- 之后的结果。所以：a + b = (a ^ b) + (a & b << 1)
- 令a' = a ^ b, b' = (a & b) << 1
- 可以知道，这个过程是在模拟加法的运算过程，进位不可能
- 一直持续，所以b最终会变为0。因此重复做上述操作就可以
- 求得a + b的值。

## 冒泡排序

```python

def bubbleSort(relist):
    len_ = len(relist)
    for i in range(len_):
        for j in range(0, len_ - i - 1):
            if relist[j] > relist[j + 1]:
                relist[j + 1], relist[j] = relist[j], relist[j + 1]
    return relist


# print(bubbleSort([1, 5, 2, 6, 9, 3]))


def bubbleSort2(inlist):
    len_ = len(inlist)
    for i in range(len_):
        for j in range(len_ - i - 1):
            if inlist[j] > inlist[j + 1]:
                inlist[j + 1], inlist[j] = inlist[j], inlist[j + 1]
    return inlist


# print(bubbleSort2([1, 5, 2, 6, 9, 3]))

```

## 快速排序

```python

def quickSort(array):
    if len(array) < 2:
        return array
    else:
        pivot = array[0]
        less = [i for i in array[1:] if i < pivot]
        greater = [j for j in array[1:] if j > pivot]
        return quickSort(less) + [pivot] + quickSort(greater)


print(quickSort([1, 5, 2, 6, 9, 3]))

# 快排 分片的思想+递归的思想，这是取了第一个为基准值，栈高为O(log(n)),栈长O(n),所以运行时间为栈高x栈长，也就是算法平均运算时间为O(nlog(n))
def quickSort2(array):
    if len(array) < 2:
        return array
    else:
        piovt = array[0]
        less = [i for i in array[1:] if i< piovt]
        greater = [j for j in array[1:] if j>piovt]
        return quickSort2(less) + [piovt] + quickSort2(greater)

print(quickSort2([1, 5, 2, 6, 9, 3]))

```

## 范围内的质数

```python

def func():
    res = []
    for i in range(1, 101):
        if compute(i):
            pass
        else:
            res.append(i)
    return res


def compute(value):
    flag = False
    if value <= 2:
        return False
    for i in range(2, value - 1):
        r = value % i
        if not r:
            flag = True
    return flag

print(func())

```

## 二叉数遍历

```python

class BinaryTreeNode(object):
    def __init__(self, data=None, left=None, right=None):
        self.data = data
        self.left = left
        self.right = right


class BinaryTree(object):
    """docstring for BinaryTree"""

    def __init__(self, root=None):
        self.root = root

    def is_empty(self):
        return self.root == None

    def preOrder(self, this_Node):
        if this_Node == None:
            return
        print(this_Node.data)
        self.preOrder(this_Node.left)
        self.preOrder(this_Node.right)

    def inOrder(self, this_Node):
        if this_Node == None:
            return
        self.inOrder(this_Node.left)

        print(this_Node.data)
        self.inOrder(this_Node.right)

    def postOrder(self, this_Node):
        if this_Node == None:
            return
        self.postOrder(this_Node.left)
        self.postOrder(this_Node.right)
        print(this_Node.data)

    def levelOrder(self, this_Node):
        if this_Node == None:
            return
        _queue = []
        _queue.append(this_Node)
        while _queue:
            node = _queue.pop(0)
            print(node.data)
            if node.left != None:
                _queue.append(node.left)
            if node.right != None:
                _queue.append(node.right)

    def deep(self, root):
        if not root:
            return
        print(root.data)
        self.deep(root.left)
        self.deep(root.right)

    def deepTree(self, root):
        if root == None:
            return 0
        ld = self.deepTree(root.left)
        rd = self.deepTree(root.right)
        return max(ld, rd) + 1


n1 = BinaryTreeNode(data="D")
n2 = BinaryTreeNode(data="E")
n3 = BinaryTreeNode(data="F")
n4 = BinaryTreeNode(data="B", left=n1, right=n2)
n5 = BinaryTreeNode(data="C", left=n3, right=None)
root = BinaryTreeNode(data="A", left=n4, right=n5)

bt = BinaryTree(root)

bt = BinaryTree(root)
# print('先序遍历')
# bt.preOrder(bt.root)
# print('中序遍历')
# bt.inOrder(bt.root)
# print('后序遍历')
# bt.postOrder(bt.root)
bt.levelOrder(root)

# print bt.deepTree(bt.root)

```

## 斐波那契数列

```python

def flb(num):
    result = None
    n, a, b = 0, 0, 1
    while n < num:
        print(b)
        result = b
        a, b = b, a + b
        n += 1
    return result


a = flb(6)
print(a)

```

## 跳台阶

```python

# def jump_floor(number):
#     if number <= 2:
#         return number
#     prev, curr = 1, 2
#     for _ in range(3, number + 1):
#         prev, curr = curr, prev + curr
#         print(curr)
#     return curr


# print(jump_floor(5))


def jump(time):
    if time <= 2:
        return time
    a, b = 1, 2
    for i in range(3, time + 1):
        a, b = b, a + b
    return b


# print(jump(6))


# def jumpm(time):
#     if time == 0:
#         return 0
#     return 2 ** (time - 1)
#
#
# fff = lambda a: a if a <= 2 else (fff(a - 1) + fff(a - 2))
#
# print(fff(6))

```