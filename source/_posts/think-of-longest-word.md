---
title: “最长子序列问题” 思考
date: 2019-01-19 14:42:03
tags: [think]
categories: [leetcode]
---

> Longest Word in Dictionary through Deleting

"最长子序列问题" 思考
<!-- more -->
```

> Longest Word in Dictionary through Deleting

> Given a string and a string dictionary, find the longest string in the dictionary that can be formed by deleting some characters of the given string. If there are more than one possible results, return the longest word with the smallest lexicographical order. If there is no possible result, return the empty string.

Example 1:

Input:
s = "abpcplea", d = ["ale","apple","monkey","plea"]

Output:
"apple"

Example 2:

Input:
s = "abpcplea", d = ["a","b","c"]
Output:
"a"

> Note:

> All the strings in the input will only contain lower-case letters.
> The size of the dictionary won't exceed 1,000.
> The length of all the strings in the input won't exceed 1,000.
```

#### 双指针的方式解决

  ```python
  class Solution:
      def findLongestWord(self, s, d):
          """
          :type s: str
          :type d: List[str]
          :rtype: str
          """
          def is_valid(s, word):
              i, j = 0, 0
              while i < len(s) and j < len(word):
                  if s[i] == word[j]:
                      j += 1
                  i += 1
              return j == len(word)

          longest = ''
          for word in d:
              l1, l2 = len(longest), len(word)
              if l1 > l2 or (l1 == l2 and longest < word):
                  continue
              if is_valid(s, word):
                  longest = word

          return longest
  ```

#### 问题描述里有很多 python 的高级写法

- [LINK](https://leetcode.com/problems/longest-word-in-dictionary-through-deleting/discuss/99590/Short-Python-solutions)

```python
def findLongestWord(self, s, d):
  def isSubsequence(x):
      it = iter(s)
      return all(c in it for c in x)
  return max(sorted(filter(isSubsequence, d)) + [''], key=len)
```
- `fliter` 函数
  - 过滤序列，过滤掉不符合条件的元素，返回符合条件的元素组成的新序列，新序列是一个迭代器
  - 接受两个参数，第一个是过滤函数，第二的是可迭代对象
  - 语法 `filter(function, iterable)`
  - 例子
    ```py
    def is_odd(x):
        return x%2 == 1
    list2 = filter(is_odd, [1, 3, 5, 6, 7, 8])
    print(list(list2))  # out [1, 3, 5, 7]
    ```
  - 过滤函数对每个元素进行条件判断，返回 bool 值，
  - 换一种写法
    ```py
    list1 = [1, 3, 5, 6, 7, 8]
    list2 = []
    for x in list1:
      if x%2 == 1:
        list2.append(x)
    print(list2)  # out [1, 3, 5, 7]
    ```
- `all` 函数
  - 判断可迭代对象中所有元素是否为 `TRUE`，如果是放回 `True`，不是返回 `False`
  - 这里 `TRUE` 的定义是除 0、空（None）、False、'' 都算 `TRUE`
  - 元素为 0 个的可迭代对象返回 `True`
  - 语法 `all(iterable)`
  - 使用
    ```py
    all([1, 2, 3]) # True
    all([1, 2, 0]) # False
    ```
  - 等同于
    ```py
    def all(iterable):
        for element in iterable:
            if not element:
                return False
        return True
    ```
  - 类似的 `any` 函数，任意元素为 `TRUE` 就返回 `True`
- `sorted` 函数
  - 对可迭代序列排序生成新序列
-  `max` 函数
  - 取出序列中最大的元素
- 流程分析
  ```
  it = iter(s)  # 将字符串转换为可迭代序列 it = ('a','b','p','c','p','l','e','a')
  #  x = 'ale'
  (c in it for c in x) # c 元素同时按在 x 中的顺序同时出现在 it 和 x 中，('a', 'l', 'e')
  all(('a', 'l', 'e'))  # 上一步每个元素都不为空时返回 True
  d = ["ale","apple","monkey","plea"]
  filter(isSubsequence, d) # 对 d 进行过滤，('ale', 'apple', 'plea')
  sorted(('ale', 'apple', 'plea')) # 上一步的结果进行排序，字典顺序
  # + ['']  处理为找的情况
  max(('ale', 'apple', 'plea', ''), key=len) # 根据元素长度取出最长的元素
  # 'apple'
  ```

---
- 不使用 `sorted`
  ```py
  def findLongestWord(self, s, d):
      def isSubsequence(x):
          it = iter(s)
          return all(c in it for c in x)
      return min(filter(isSubsequence, d) + [''], key=lambda x: (-len(x), x))

  # min(((-3, 'ale'), (-5, 'apple'),(-4, 'plea')))
  # 'apple'
  # 巧妙地运用 min 函数对元组的效果
  ```
- 循环替代 `filter` `max`
  ```py
  def findLongestWord(self, s, d):
      best = ''
      for x in d:
          if (-len(x), x) < (-len(best), best):
              it = iter(s)
              if all(c in it for c in x):
                  best = x
      return best
  ```
- 先进行排序
  ```py
  def findLongestWord(self, s, d):
      def isSubsequence(x):
          it = iter(s)
          return all(c in it for c in x)
      d.sort(key=lambda x: (-len(x), x))
      return next(itertools.ifilter(isSubsequence, d), '')
  ```
- 先排序，在循环
  ```py
  def findLongestWord(self, s, d):
      for x in sorted(d, key=lambda x: (-len(x), x)):
          it = iter(s)
          if all(c in it for c in x):
              return x
      return ''
  ```
- `heapq.heapify` 替换 `sorted`
  ```py
  def findLongestWord(self, s, d):
      heap = [(-len(word), word) for word in d]
      heapq.heapify(heap)
      while heap:
          word = heapq.heappop(heap)[1]
          it = iter(s)
          if all(c in it for c in word):
              return word
      return ''
  ```
