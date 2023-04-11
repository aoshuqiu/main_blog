---
title: leetcode 336 回文对
date: 2020-08-06 20:26:17
tags: ["leetcode","算法"]
categories: "学习"
mathjax: true
---

# 回文对问题



## 题目

>#### 
>
>给定一组 **互不相同** 的单词， 找出所有**不同** 的索引对`(i, j)`，使得列表中的两个单词， `words[i] + words[j]` ，可拼接成回文串。
>
> 
>
>**示例 1：**
>
>```
>输入：["abcd","dcba","lls","s","sssll"]
>输出：[[0,1],[1,0],[3,2],[2,4]] 
>解释：可拼接成的回文串为 ["dcbaabcd","abcddcba","slls","llssssll"]
>```
>
>**示例 2：**
>
>```
>输入：["bat","tab","cat"]
>输出：[[0,1],[1,0]] 
>解释：可拼接成的回文串为 ["battab","tabbat"]
>```

<!-- more -->

## 暴力做法

两重循环枚举字符串组合，判断是否能构成回文串。时间复杂度为 $O(n^2\times m)$ ，其中n为字符串数量，m为字符串平均长度。测试用例中的字符串数目都很多，因此这个时间复杂度不能满足题目要求。



## 枚举前缀与后缀

**思路及算法 **

如果两个字符串$ s_1 $和$s_2$，$s_1 + s_2$是一个回文串，那么对于其中较长的字符串$ s$可以分为两个部分，一部分是一个回文串，另一部分是较短的字符串的翻转。其中空串也为回文串。

也就是说，枚举每一个字符串$k$令其为$s_1$,$s_2$中较长的那一个，枚举字符串k的包括空串的每一个前缀与后缀，判断其是否为回文串，如果是回文串，就判断剩余部分的翻转是否在字符串序列中。

其中字符串搜索有两种方式：字典树与哈希表

**字典树 **

一个树型结构，每个字符输入对应一个有向边，每个节点对应一个字符串。根节点为空串。

``` python
#树的节点，ch代表节点接收一个字符的出边，值为指向的节点序号
#flag: 节点代表的字符串的索引
class Node:
    def __init__(self):
        self.ch = [0] * 26
        self.flag = -1

class Solution:
    def palindromePairs(self, words: List[str]) -> List[List[int]]:
        #初始化字典树
        tree = [Node()]
		
        #s:要插入的字符串
        #index: s对应的索引
        def insert(s: str, index: int):
            length = len(s)
            add = 0
            for i in range(length):
                x = ord(s[i]) - ord("a")
                #不存在这个字符的子节点:新建一个节点并指向它
                if tree[add].ch[x] == 0:
                    tree.append(Node())
                    tree[add].ch[x] = len(tree) - 1
                #移动到下一个节点继续搜寻
                add = tree[add].ch[x]
            tree[add].flag = index
        
        #s:要寻找的字符串
        #left:s最左字符索引
        #right:s最右字符索引
        def findWord(s: str, left: int, right: int) -> int:
            add = 0
            for i in range(right, left - 1, -1):
                x = ord(s[i]) - ord("a")
                if tree[add].ch[x] == 0:
                    return -1
                add = tree[add].ch[x]
            return tree[add].flag
        
        def isPalindrome(s: str, left: int, right: int) -> bool:
            length = right - left + 1
            return length < 0 or all(s[left + i] == s[right - i] for i in range(length // 2))
        
        n = len(words)
        for i, word in enumerate(words):
            insert(word, i)
        
        ret = list()
        for i, word in enumerate(words):
            m = len(word)
            for j in range(m + 1):
                if isPalindrome(word, j, m - 1):
                    leftId = findWord(word, 0, j - 1)
                    if leftId != -1 and leftId != i:
                        ret.append([i, leftId])
                if j and isPalindrome(word, 0, j - 1):
                    rightId = findWord(word, j, m - 1)
                    if rightId != -1 and rightId != i:
                        ret.append([rightId, i])

        return ret
```



**哈希表 **

python中通过dict类型作为哈希表

``` python
class Solution:
    def palindromePairs(self, words: List[str]) -> List[List[int]]:

        def findWord(s: str, left: int, right: int) -> int:
            return indices.get(s[left:right+1], -1)
        
        def isPalindrome(s: str, left: int, right: int) -> bool:
            return (sub := s[left:right+1]) == sub[::-1]
        
        n = len(words)
        indices = {word[::-1]: i for i, word in enumerate(words)}
        
        ret = list()
        for i, word in enumerate(words):
            m = len(word)
            for j in range(m + 1):
                if isPalindrome(word, j, m - 1):
                    leftId = findWord(word, 0, j - 1)
                    if leftId != -1 and leftId != i:
                        ret.append([i, leftId])
                if j and isPalindrome(word, 0, j - 1):
                    rightId = findWord(word, j, m - 1)
                    if rightId != -1 and rightId != i:
                        ret.append([rightId, i])

        return ret
```



**复杂度分析**

时间复杂度为$O(n \times m^2)$，其中需要$O(m^2)$来判断一个字符串所有的前缀与后缀是否是回文串，并且$O(m^2)$地判断剩余部分是否在字符串序列中出现。

